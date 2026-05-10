# Column-Scoped Inference: Preventing the AI from Proposing Irrelevant Charts

**Series:** LLM-Driven Data Visualisation — Part 3 of 7

---

Give an LLM a 31-column dataset and ask “what’s interesting?” and it will generate noise. Wide schemas explode the hypothesis space. This is how to implement column-scoped inference: constraining the AI's analytical horizon to specific subsets of data before it sees the summary.

## Who this is for

You are using LLMs to propose charts or hypotheses over wide datasets and you keep getting plausible-but-useless chart packs.

## What you’ll learn

- Why wide schemas explode the hypothesis search space
- How column-scoped inference constrains the model without prescribing conclusions
- How to structure “one pack per question” workflows that remain auditable

## Constraints

- The model must only see a scoped evidence set per analytical question
- Domain abstraction applies (no client datasets named; use publish-safe equivalents)

Giving an LLM a 31-column dataset and asking "what is interesting?" is a reasonable question with an unreasonable answer surface.

The model will generate hypotheses. Some will be analytical gold. Most will be plausible combinations of columns that produce charts no one needed — because with 31 columns, the combination space is large and the model has no way to know which subset of that space maps to the actual business question you are trying to answer.

The result is a broad chart pack that covers the dataset rather than the question.

Column-scoped inference is the fix. You tell the pipeline which columns are relevant to a specific analytical question before the LLM sees the data summary. The model reasons within a constrained space — not because it has been told what conclusion to reach, but because it has been given the right evidence set to start from.

## What happens without scoping

Consider an enterprise incident dataset with 31 columns: `number`, `opened_at`, `closed_at`, `state`, `priority`, `category`, `subcategory`, `assignment_group`, `assigned_to`, `description`, `resolution_notes`, `sla_breach`, `reopen_count`, `close_code`, and seventeen more.

[NEEDS_DATA: actual output from a broad inference run over the 31-column incident dataset — chart titles produced and whether they address a coherent analytical question]

The model's Goal Exploration phase (Part 2) will generate hypotheses that span the full column space. Some will address priority distribution. Some will address assignment group load. Some will mix resolution time with category in ways that are statistically valid but analytically disconnected from any real question.

The self-critique pass (Part 4) catches some of this drift. But it is easier to prevent unfocused hypotheses than to repair them.

## The multi-pack pattern: one config per analytical question

The solution is to split one broad run into multiple focused runs, each scoped to the columns that matter for a specific question.

For a wide incident dataset, three analytical questions map to three distinct column sets:

**Question 1 — Taxonomy health:** Is the incident taxonomy being applied consistently? Which categories dominate? Are subcategories distributed as expected?
```yaml
inference:
  columns: [category, subcategory, priority, close_code]
  prompt: "Analyse the taxonomy and classification patterns in this incident dataset."
```

**Question 2 — Operational routing:** Where are incidents being assigned, and how does that map to resolution time?
```yaml
inference:
  columns: [assignment_group, assigned_to, priority, sla_breach, reopen_count]
  prompt: "Analyse assignment routing, SLA performance, and rework patterns."
```

**Question 3 — Data quality:** Which fields have high null rates, inconsistent values, or patterns suggesting poor data entry?
```yaml
inference:
  columns: [state, close_code, resolution_notes, reopen_count, category]
  prompt: "Identify data quality signals — null patterns, inconsistent close codes, and incomplete resolution records."
```

Three runs. Three focused packs. Each one addresses one question clearly.

[NEEDS_DATA: actual chart titles and insight statements from three column-scoped configs — showing the difference in analytical focus vs a broad run]

## How scoping works in the pipeline

The `inference.columns` parameter filters the data summary before it is sent to the LLM. The model sees only the statistical profile of the specified columns — cardinalities, distributions, null rates, top values. The remaining 27 columns do not appear in the summary at all.

This has a secondary benefit: it reduces column hallucination (described in Part 2). With 6 columns in scope instead of 31, the probability that the model invents a column name it "remembers" from similar datasets drops significantly.

```yaml
mode: infer_visualisations

inference:
  top_k: 5
  columns: [category, subcategory, priority, close_code]
  prompt: "Analyse the taxonomy and classification patterns in this incident dataset."
  llm:
    model: "qwen3.5:9b"
    provider: "ollama"
```

The `top_k: 5` parameter sets the maximum number of charts. With 4 columns in scope, the model typically generates 4-5 focused charts rather than 5 unfocused ones.

## The trade-off: three runs vs one

The cost is parallelism. Three separate runs means three times the LLM calls and three times the wall time.

[NEEDS_DATA: total runtime for the three-pack vs a single broad run over the same dataset — showing the trade-off concretely]

For most analytical workflows, this trade-off is acceptable because the runs are independent and can be executed in parallel. The pipeline is also fast enough at local inference speeds that three focused runs complete in less time than one broad run followed by manual culling of irrelevant charts.

The practical heuristic: use broad inference for dataset exploration (when you have no prior hypothesis about what matters). Use column-scoped inference once you know which analytical questions are worth answering.

## When to use each approach

| Scenario | Approach |
|----------|----------|
| First look at an unfamiliar dataset | Broad inference — discover what is in the data |
| Known business question, known column set | Column-scoped inference — focused pack, lower hallucination |
| Multiple distinct stakeholders, different questions | Multi-pack — one focused run per audience |
| Fewer than 10 columns | Broad inference — scoping adds no value |

The Seattle weather dataset (6 columns, Part 2) uses broad inference because there are too few columns for scoping to matter. A wide incident dataset (31 columns) uses the multi-pack pattern because each analytical question maps to a distinct subset.

## The lesson

Constraining the question before you ask the AI to answer it is not a limitation of the tool. It is a discipline that produces better analysis.

An analyst presenting to a board does not say "here is everything the data contains." They say "here is what the data tells us about the question we are trying to answer." Column scoping is how you apply that discipline to an AI pipeline.

## Limits / when not to use

- Column scoping reduces noise, but it can hide cross-column interactions if you scope too aggressively.
- The approach assumes you can define questions up front; if the goal is discovery, start broader.
- Scoped packs still need validation and telemetry; scoping alone is not production hardening.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; [NEEDS_DATA: dataset used (public or anonymised), scoped column sets used]
- Dataset class: public tabular dataset for publish-safe screenshots; anonymised incident dataset for internal benchmarking
- Non-reproducible from this article: any client-specific schema conventions or internal packs

---
