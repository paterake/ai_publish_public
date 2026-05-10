# From Dataset to Stakeholder Pack: A Strict Vega-Lite Workflow That Repairs Itself

**Series:** LLM-Driven Data Visualisation — Part 7 of 7

---

Executives don’t want charts. They want a pack. The pack must be repeatable, explainable, and fast to regenerate. This is how the pipeline outputs a complete Vega-Lite analyst pack directly from config.

## Key takeaways

- A stakeholder pack is an output product: multiple charts, narratives, and an audit trail, regenerated from config.
- Strict validation is the trust boundary: bad column references and invalid specs fail before any artefact is written.
- Repairs are targeted: fix only the failed Vega-Lite spec using the schema error, without rewriting the analysis plan.

## Who this is for

You are building or operating an AI-enabled data workflow and you care about reliability, auditability, and repeatability more than demos.

## What you’ll learn

- The key design decisions that made the final implementation work
- The turning points (attempt → failure → decision → proof) that justify the architecture
- The limits and where the approach does not apply

## Constraints

- Local-first / zero data egress where relevant
- Outputs must be auditable (sources, artifacts, manifests)

There is a quiet but important distinction in AI charting:

- A demo system produces a chart once.
- A production system produces a chart pack repeatedly, with traceability, validation, and bounded failure modes.

This article documents the complete workflow in the platform’s visualisation module (called `elt_llm_visualise` in this codebase): a config-driven pipeline that asks an LLM to propose an analyst pack, validates every spec deterministically, and repairs only what fails — leaving the rest of the plan intact.

The design goal is publication and stakeholder use. A silent wrong chart is worse than a loud correct failure.

## The problem: LLM charting breaks in predictable ways

Modern models describe datasets well. The failure modes when generating charts are also predictable:

- Plotting libraries require imperative code that is hard to generalise across datasets
- LLMs hallucinate field names — they "remember" that datasets like yours often have a `season` column, a `month_name` derived field, or a `percentage` pre-computed field, and they propose charts based on that memory rather than the actual schema
- The easiest patch for a chart failure is to silently modify the output — which can change what the chart means

The pipeline is designed around the inverse: fail loudly on invalid specs, repair only what failed, and produce an audit trail for every run.

## The architecture: the LLM proposes, the runtime proves

The central design decision is the trust boundary:

- **The LLM proposes:** analytical hypotheses, chart specs, transformation logic, insight narrative
- **The runtime proves:** that column references are valid, transformations are coherent, and Vega-Lite specs conform to the v5 schema before any HTML is written

This separation is what makes the pipeline scale. The LLM is powerful at analytical framing. The runtime is deterministic at execution. Neither crosses into the other's domain.

## End-to-end: from config to stakeholder report

### 1) Define the dataset and the question in YAML

You provide the dataset (local file or URL), the analytical question (prompt), and rendering preferences. You do not specify chart types.

```yaml
mode: infer_visualisations

dataset:
  path: "https://cdn.jsdelivr.net/npm/vega-datasets/data/iris.json"
  format: "json"

inference:
  top_k: 5
  enable_self_critique: true
  prompt: "Create a publication-quality analysis pack explaining how iris species differ across measurements."
  context: |
    Canonical dataset for visualisation development.
    Focus on species comparisons and relationships between numeric measures.

output:
  renderer: "vega_lite"
  renderer_options:
    strict: true
```

### 2) The runtime summarises the dataset without sending raw rows

The pipeline reads the dataset and produces a compact statistical profile: column types, cardinalities, null rates, top values, distribution statistics, date ranges. This summary is the LLM's input. Individual rows are never sent to the model.

For a dataset the size of iris (150 rows, 5 columns), this step completes in milliseconds.

### 3) Goal Exploration: the LLM commits to analytical intent

Before proposing any charts, the model generates 3-5 analytical hypotheses — specific questions the dataset could answer and relationships worth investigating. These hypotheses constrain the chart planning: every chart must address a stated hypothesis.

This step is what prevents the model from generating plausible-but-generic charts. It must articulate what it is trying to prove before it is allowed to propose evidence.

### 4) Prompt guardrails constrain what the model can propose

Two guardrails are active during chart planning:

**A curated "recipe card"** — a short set of Vega-Lite schema rules and example patterns that the model must follow. Optimised for the most common failure points: stacking, mark types, encoding types, and field references. Stored as configuration, not hardcoded.

**A transformation-aware field rule** — if the runtime aggregates the data (group-by + aggregate), the model can only reference post-transform column names in the Vega-Lite spec. Pre-transform columns are out of scope after aggregation. This eliminates a large class of specs that look correct but fail at render time.

### 5) Deterministic transformation execution

The plan's transformation block is executed in Pandas with strict validation:

- `group_by` columns must exist in the dataset
- `value_column` must be numeric for aggregate operations (sum, mean, std, etc.)
- `filter` expressions must be parseable and valid
- `top_n` produces an automatic "Other" bucket for single-dimension groupings, preserving total values

This is where credibility is established: it is no longer "the model said so" — it is "the data supports it, computed deterministically."

### 6) Vega-Lite validation in strict mode

Every chart spec passes through the Vega-Lite v5 schema validator before rendering. Strict mode applies two rules:

- **Schema-safe fixes are allowed:** unwrapping invalid shapes (for example, `{"stack": {"value": "normalise"}}` → `"normalise"`) where the intent is unambiguous
- **Semantic rewrites are blocked:** if a chart cannot be rendered without changing what it means, it fails loudly rather than being silently modified

A spec that invents a column name fails at transformation validation (step 5). A spec with invalid Vega-Lite syntax fails at schema validation (step 6). Neither reaches the renderer.

### 7) Targeted repair for renderer failures

When a chart spec fails schema validation, the runtime triggers a single targeted repair: it sends the failed spec and the schema error back to the LLM and asks for a corrected `vega_lite_spec` only. The insight statement, narrative, and transformation plan remain unchanged.

This is a deliberate design choice:
- **Minimises drift** — the analytical content (what the chart is saying) does not change during repair; only the rendering definition is corrected
- **Bounds retries** — one repair attempt per chart, not an open-ended retry loop
- **Converts schema errors into structured repair prompts** — the LLM receives an exact error message, not a vague "please fix this"

### 8) Shareable output artefacts

A complete run produces:

| Artefact | Purpose |
|----------|---------|
| Per-chart HTML files | Self-contained charts for embedding or individual review |
| `index.html` | Navigation index for the full pack |
| `report.html` | Combined analyst pack: charts + insight statements + narratives + recommended actions |
| `run_manifest.json` | Audit trail: run ID, config fingerprint, dataset fingerprint, model, LLM call counts, output file list, timestamps |

`report.html` is designed for direct stakeholder use — no dependencies, no build step, open in a browser.

To disable self-critique for faster iteration: set `enable_self_critique: false` in the config. To use Plotly instead of Vega-Lite: change `output.renderer` to `plotly` (covers chart types like treemap and sankey, which Vega-Lite strict mode does not yet support).

## Why this approach is better than code generation

| Concern | Code generation | This pipeline |
|---------|----------------|---------------|
| Validation before execution | Not possible without running the code | Full schema validation before rendering |
| Failure mode | Python exception at runtime | Structured error at validation, before any chart renders |
| Silent semantic modification | Common (patch the output) | Blocked in strict mode |
| Reproducibility | Depends on model state and random seed | Config fingerprint + dataset fingerprint in manifest |
| Repair scope | Re-run the whole chart | Repair only the failed spec; insight and transformation preserved |

## The broader lesson

The future of AI visualisation is not a model that guesses chart code and hopes the execution works.

It is a governed workflow: the model proposes a plan, the runtime executes deterministically, the schema validator checks the output before it ships, and failures trigger targeted repair rather than silent mutation.

That is what turns an AI charting demo into a pipeline you can trust to produce stakeholder reports reliably.

## Limits / when not to use

- Packs optimise for repeatability; they are not ideal for exploratory, iterative notebook-style work.
- Narrative quality depends on the evidence available; without reliable summaries and citations, packs become decorative.
- Publishing requires public-safe datasets for screenshots and outputs; do not leak client context.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; `inference.top_k=5`; `enable_self_critique=true`; pack output writes manifest + index + per-chart Vega-Lite specs; renderer `strict=true`
- Dataset class: Seattle weather (public Vega dataset; CSV)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---
