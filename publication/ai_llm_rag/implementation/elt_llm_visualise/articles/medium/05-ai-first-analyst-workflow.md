# The AI-First Analyst Workflow: The LLM Proposes, the Runtime Proves

**Series:** LLM-Driven Data Visualisation — Part 5 of 7

---

The mistake is treating LLM visualisation as a chart generator. The value is an analyst workflow: intent, evidence, critique, outputs. This is the end-to-end architecture of an AI-first loop that produces artifacts you can audit.

## Key takeaways

- The LLM proposes; the runtime proves: validation, critique, and deterministic execution sit between intent and output.
- Trust comes from artefacts: a shareable report plus a manifest that records what ran, what data was used, and what was produced.
- The workflow changes the job: analysts define questions; the pipeline generates and validates the pack.

## Who this is for

You are building or operating an AI-enabled data workflow and you care about reliability, auditability, and repeatability more than demos.

## What you’ll learn

- The key design decisions that made the final implementation work
- The turning points (attempt → failure → decision → proof) that justify the architecture
- The limits and where the approach does not apply

## Constraints

- Local-first / zero data egress where relevant
- Outputs must be auditable (sources, artifacts, manifests)

There is a version of "AI-assisted analysis" that does not change how you work. You write SQL. You export a CSV. You open a notebook. At some point you ask an AI assistant to suggest a chart type, and it says "bar chart." You were already going to use a bar chart. Nothing changed.

The analyst workflow in the platform’s visualisation module (called `elt_llm_visualise` in this codebase) is built on a different premise: the AI does the analytical work, and the runtime proves that the work is correct. The analyst defines the question. The pipeline answers it.

This article describes the end-to-end workflow — from question to shareable report — and explains why each step is designed the way it is.

## The design principle: a trust boundary between proposal and proof

Every AI-first system has an implicit trust boundary: a line between what the AI is allowed to decide and what the system verifies independently. In most AI charting tools, the boundary is permissive — the AI decides, and if the output looks reasonable, it ships.

In this pipeline, the boundary is explicit:

- **The LLM proposes:** analytical hypotheses, chart types, transformation logic, insight statements, narrative, and recommended actions — all expressed as structured JSON
- **The runtime proves:** that referenced columns exist, that transformations are coherent, that the Vega-Lite spec is schema-valid, and that the output artefacts are reproducible

The LLM is powerful at the proposal side. The runtime is deterministic on the proof side. Neither side crosses into the other's domain.

## Step 1: Define the question, not the charts

You provide three things in a YAML config:

- **The dataset** — local file path or URL; CSV, JSON, JSONL, or Excel
- **The analytical prompt** — the question you want answered, in plain language
- **Optional: column scope and context** — what the dataset represents; which columns are relevant to this question

```yaml
mode: infer_visualisations

dataset:
  path: "https://cdn.jsdelivr.net/npm/vega-datasets/data/seattle-weather.csv"
  format: "csv"

inference:
  top_k: 5
  enable_self_critique: true
  prompt: |
    Create a publication-quality analysis pack explaining seasonality,
    precipitation patterns, and temperature trends in Seattle weather.
  context: |
    This is a time-series dataset covering 2012-2015 Seattle weather observations.
    Prefer temporal charts and layered reference lines where appropriate.

output:
  renderer: "vega_lite"
  renderer_options:
    strict: true
```

You do not specify chart types. You do not write transformation logic. You state the question.

## Step 2: The pipeline profiles the dataset

Before involving the LLM, the pipeline reads the dataset and builds a compact statistical summary:

- Column types (numeric, categorical, temporal)
- Cardinalities and null rates
- Distribution statistics (mean, std, quartiles for numerics; top values and frequencies for categoricals)
- Date ranges for temporal fields

This summary is the LLM's primary input. The pipeline is designed to avoid sending raw rows to the model — the summary gives the model enough statistical grounding to reason about the data without exposing individual records.

For a 1,461-row CSV, this step completes in under a second.

## Step 3: Goal Exploration — the LLM writes an analysis brief

Before proposing any charts, the LLM is asked to generate 3-5 analytical goals: specific questions the dataset could answer, relationships worth investigating, and patterns that might surface useful insights.

This step is what distinguishes the workflow from "LLM picks chart types." The model must commit to analytical intent — in writing, in structured form — before it is allowed to propose any visualisations. The goals constrain the chart planning: every chart must address a stated goal.

For the Seattle weather dataset, Goal Exploration produces hypotheses in the form of: "Is there a seasonal pattern in the proportion of different weather types?" and "Does temperature range (max - min) vary predictably with precipitation?" — targeted, verifiable questions rather than generic "show me the data" prompts.

## Step 4: The LLM plans a structured specification per chart

For each goal, the LLM returns a `VisualisationSpec` — a structured JSON object containing:

- **Insight surface:** `statement` (finding), `narrative` (explanation), `action` (recommendation), `importance` (0-1 priority score)
- **Transformation plan:** `filter`, `group_by`, `aggregate`, `value_column`, `sort`, `top_n` — the complete data preparation logic, expressed as a declarative spec rather than code
- **Vega-Lite spec:** the declarative chart definition, validated against the Vega-Lite v5 schema

The LLM returns JSON. It does not generate Python. It does not use `exec()`. The transformation plan is executed deterministically in Pandas by the runtime.

## Step 5: Self-critique before rendering (optional)

With `enable_self_critique: true`, the pipeline passes the complete plan back to the LLM for review before any chart is rendered. The model evaluates each spec for readability, correctness, and actionability — and corrects what it finds problematic.

This is a lightweight quality gate. It adds one LLM call but catches the most common failure mode: a chart that is technically valid but analytically misleading (wrong chart family for the data type, too many categories, a narrative that does not match the visual).

## Step 6: Deterministic execution and Vega-Lite rendering

The runtime executes each transformation plan in sequence:

1. Apply filters to the dataset
2. Group by specified columns, aggregate the value column
3. Sort and apply top-N with automatic "Other" bucket for single-dimension groupings
4. Validate the resulting DataFrame against the column references in the Vega-Lite spec
5. Validate the Vega-Lite spec against the v5 schema in strict mode
6. Inject the transformed data into the spec and render HTML

If a column reference is wrong, step 4 fails before any rendering runs. If the Vega-Lite spec is invalid, step 5 fails with a human-readable schema error. Neither failure is silent.

## Step 7: Shareable artefacts

The Seattle weather run (2026-04-10, 1,461 rows, qwen3.5:9b, 6m28s) produced:

- Five chart HTML files, one per hypothesis (self-contained, no external dependencies)
- `index.html` — navigation index for the pack
- `run_manifest.json` — the audit trail: run ID, config fingerprint, dataset fingerprint, model, LLM call counts, token estimates, output file list, start and finish timestamps

`report.html` — the combined analyst pack — contains each chart alongside its insight statement, narrative, and recommended action. It is designed to be shared directly with stakeholders without any additional formatting.

[NEEDS_DATA: screenshot of report.html from the Seattle weather run — showing at least one chart with its insight panel visible]

## How the workflow runs in practice

The entire workflow — from a YAML config and a dataset to a shareable analyst pack with an audit trail — runs as a single command. The details are intentionally omitted here because the implementation lives in a private repository; what matters for reproducibility is the configuration, the dataset fingerprint, and the run manifest written alongside the outputs.

## What this enables

The workflow changes the analyst's job from chart production to question definition. The analyst writes the prompt. The pipeline proposes, validates, and renders. The analyst reviews the output, refines the prompt if needed, and shares the report.

The charts are not the product. The questions are. The pipeline answers them reliably enough that the analyst can trust the output rather than verify each chart individually.

That is the goal of AI-first tooling: not AI that assists humans doing manual work, but AI that takes over the repeatable analytical work so humans can focus on the questions worth asking.

## Limits / when not to use

- This workflow is not a replacement for deep statistical analysis; it is a structured path to a stakeholder pack.
- If prompts or configs are treated as magic, the system will drift; you still need ownership of definitions and constraints.
- The workflow assumes auditability is valued; if the org only wants “pretty charts”, the extra discipline will be resisted.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; `inference.top_k=5`; `enable_self_critique=true`; `temperature=0.1`; HTML report output with manifest/index enabled; Vega-Lite renderer `strict=true`
- Dataset class: Seattle weather (public Vega dataset; 1,461 rows; 2012–2015)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---
