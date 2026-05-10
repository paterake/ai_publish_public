# Production-Grade AI Visualisation: Self-Critique, Telemetry, and Validation

**Series:** LLM-Driven Data Visualisation — Part 4 of 7

---

A demo chart pipeline breaks the first time a column name is wrong. Production needs validation, telemetry, and a failure posture. This covers the hardening gates — self-critique, schema validation, and manifest generation — required before any AI-planned chart renders.

Here is what a broken run looks like before you add hardening gates:

```text
done_reason='length' … content='{ "visualisations": [ … "vega_lite_spec": { … "y2": { … [TRUNCATED]
```

Technically coherent. Not executable. If you do not detect and stop on this, the failure mode is silent: you get an index file and a broken pack.

## Who this is for

You are building or operating an AI-enabled data workflow and you care about reliability, auditability, and repeatability more than demos.

## What you’ll learn

- The key design decisions that made the final implementation work
- The turning points (attempt → failure → decision → proof) that justify the architecture
- The limits and where the approach does not apply

## Constraints

- Local-first / zero data egress where relevant
- Outputs must be auditable (sources, artifacts, manifests)

Getting one chart out of an LLM is a demo. Running five charts reliably, across multiple datasets, with an audit trail and bounded failure modes — that is a production pipeline.

The difference is not chart quality. A demo produces charts that look correct. A production pipeline produces charts that are correct — and proves it.

This article covers the three mechanisms that bridge the gap: schema validation (catches hallucinated column names before execution), self-critique (the LLM reviews its own plan before any chart renders), and run manifests (every execution is recorded with enough context to reproduce or debug it).

## Why LLM charting fails without governance

The first production run of the Seattle weather pipeline (2026-04-10) had five render failures — every chart the LLM proposed failed Vega-Lite schema validation. Only the `index.html` was written. Three LLM calls were made.

The second run of the same config (same date) succeeded completely — five charts, six LLM calls (including self-critique), 6m28s total.

What changed between runs was not the config or the dataset. It was the prompt engineering in the config — specifically, explicit guidance to the LLM about the Vega-Lite patterns that are and are not valid in strict mode. The first run produced specs that were plausible but not schema-valid. The second run produced specs that the validator accepted.

Both runs are recorded in the manifest directory. You can diff the configs and trace exactly why one failed and the other succeeded. That is what production governance looks like: not silent success, but traceable outcomes.

## Layer 1: schema validation at the trust boundary

The pipeline never passes raw dataset rows to the LLM. It passes a compact statistical summary. The LLM then returns a structured plan — a `VisualisationSpec` containing an insight surface, a transformation definition, and a Vega-Lite spec.

Before any transformation is executed or any chart is rendered, the runtime validates:

1. **Column references exist** — every column named in the transformation block (`group_by`, `value_column`, `filter` expressions) must exist in the actual dataset schema
2. **Transformation logic is coherent** — you cannot aggregate `sum` over a categorical column, sort by a column not in the output, or apply `top_n` without a value column
3. **Vega-Lite spec is schema-valid** — the spec is validated against the Vega-Lite v5 schema in strict mode before rendering

When strict mode is enabled, the validator applies two rules:
- **Schema-safe fixes** are permitted — for example, unwrapping an invalid nested shape like `{"stack": {"value": "normalise"}}` into the correct `"normalise"` string
- **Semantic rewrites are blocked** — if a chart cannot be rendered without changing what it means, it fails loudly

A hallucinated column name fails at step 1, before any Pandas computation runs. A schema-invalid Vega-Lite spec fails at step 3, before any HTML is written. Neither failure is silent.

## Layer 2: self-critique before rendering

When `enable_self_critique: true`, the pipeline runs an additional LLM call after the chart plan is complete and before any chart is rendered.

The model is given its own plan — all five visualisation specs — and asked to evaluate each one against three criteria:

- **Readability** — too many categories, wrong chart family for the data type, colour encoding that obscures rather than reveals
- **Correctness** — invalid aggregation, field references that do not match the post-transformation schema, mark types inconsistent with the encoding
- **Actionability** — does the narrative statement actually explain what the chart shows? Is the recommended action specific enough to be useful?

For the Seattle weather run, the self-critique pass was the sixth LLM call (after goal exploration + five chart plans). It identified and corrected at least one spec before rendering.

[NEEDS_DATA: the specific self-critique correction from the 2026-04-10 run — which chart was flagged, what the original spec contained, what the corrected spec changed]

The self-critique pass is optional (`enable_self_critique: false` skips it). The trade-off: one additional LLM call per run, approximately 30-60 seconds at local inference, in exchange for a targeted quality review before the render phase.

## Layer 3: the run manifest

Every successful and failed run writes a `run_manifest.json`. This is the audit trail.

From the successful Seattle weather run (2026-04-10):

```json
{
  "run_id": "<run_id>",
  "started_at": "2026-04-10T10:02:38+00:00",
  "finished_at": "2026-04-10T10:09:06+00:00",
  "status": "success",
  "config_fingerprint": "dee5dda4a7e1533e23eed02571dd8158290f00ac0da8ad8e9f67a17bfd3337ec",
  "model": "qwen3.5:9b",
  "dataset_rows_raw": 1461,
  "dataset_cols_raw": 6,
  "outputs_written": 6,
  "llm_counters": {
    "llm_calls": 6,
    "prompt_tokens_est": 13284,
    "output_tokens_est": 4398,
    "cost_gbp_est": 0.0,
    "llm_failures": 0
  }
}
```

The `config_fingerprint` means you can reconstruct exactly what ran. The dataset fingerprint covers the data — if the data changes, the fingerprint changes, and you have a record of which run used which version of the data.

From the failed earlier run (same date):

```json
{
  "run_id": "<run_id>",
  "status": "success",
  "error_counts": {
    "render_failure": 5
  },
  "outputs_written": 1,
  "llm_counters": {
    "llm_calls": 3,
    "llm_failures": 0
  }
}
```

Note: `status: "success"` at the run level means the pipeline completed without crashing. `error_counts.render_failure: 5` means every chart failed the Vega-Lite validator. `outputs_written: 1` — only the index was written. The pipeline ran to completion, logged every failure, and produced a manifest recording exactly what happened. No silent corruption of output.

## Why Vega-Lite beats code generation for production reports

The architectural decision underlying all three layers is the choice to use Vega-Lite's declarative spec format instead of asking the LLM to generate Python plotting code.

The standard alternative is code generation: ask the LLM to write `fig = px.bar(...)` statements, then execute them. This is common and sometimes works. The failure modes are structural:

- Generated code can execute any operation on the data — `exec()` is effectively unlimited
- Syntax errors produce Python exceptions, not schema validation errors with human-readable messages
- Code is opaque to downstream validation — you cannot check "does this code produce a correct stacked bar chart?" without running it

A Vega-Lite spec is a JSON object with a formal schema. Every field has a type, a valid value set, and a defined relationship to other fields. The validator can check all of this before the render runs. The LLM produces a constrained JSON object; the runtime proves whether it is correct.

The trade-off: Vega-Lite coverage. At the time of writing, the Vega-Lite renderer does not support treemap, sankey, or parallel_categories charts. The Plotly renderer covers all 14 chart types. The migration path is to full Vega grammar (Step 2 of the rendering roadmap), which covers everything Plotly does plus declarative validation.

## The lesson

A production AI pipeline is not one that produces correct output most of the time. It is one that produces traceable, verifiable output — where failures are explicit, reproducible, and recoverable.

Validation at the trust boundary, a self-correcting quality pass, and a deterministic audit trail are not hardening options to add after the pipeline works. They are the difference between a system you can trust and a system you have to babysit.

## Limits / when not to use

- Hardening adds friction (validation, critique, retries). For one-off charts, it may feel heavy.
- Schema validation prevents bad outputs, but it cannot guarantee that the chosen chart answers the right question.
- Reliable manifests require disciplined output roots and stable config; ad-hoc runs reduce traceability.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; `inference.top_k=5`; `enable_self_critique=true`; `max_parse_retries=2`; Vega-Lite renderer `strict=true`; HTML report output with manifest/index enabled
- Dataset class: Seattle weather (public Vega dataset; CSV)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---
