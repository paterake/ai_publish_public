# Goal Exploration: Why LLM Visualisation Should Start with Hypotheses

**Series:** LLM-Driven Data Visualisation — Part 2 of 7

---

Most AI chart tools go straight to: “pick a chart”. That produces plausible visuals with no analytical stake. This is why you must force the LLM to generate hypotheses first, committing to analytical intent before it is allowed to plan a single chart.

## Who this is for

You are experimenting with LLM-driven charting and you want results that feel like analysis, not a generic chart gallery.

## What you’ll learn

- Why “pick a chart” is the wrong first step
- How forcing hypotheses up front reduces randomness and improves relevance
- How to keep this process auditable and bounded

## Constraints

- The model must commit to analytical intent before generating chart specs
- Outputs must be explainable (hypotheses → chosen charts), not just rendered

Most "AI visualisation" tools ask the model one question: what chart should I draw?

The model returns a chart type. You get a bar chart. It is probably correct. It is almost certainly generic.

The problem is not the chart. The problem is that the model went straight to output without doing any analytical work. It did not ask what question the data should answer, which relationships were worth investigating, or whether the proposed chart would actually surface something useful. It just picked a bar chart because bar charts are common.

This article shows a different approach: making the LLM write an analysis brief before it touches a single chart spec. The pattern is called Goal Exploration, and it is the mechanism that produces targeted analysis rather than a chart buffet.

## What most implementations skip

The standard pattern for AI-driven visualisation is:

1. Send dataset description (or raw data) to LLM
2. Ask: "what charts should I create?"
3. Receive list of chart suggestions
4. Render them

The failure mode is subtle. When you ask "what charts should I create?", you are asking the model to produce output immediately. It skips the step where a competent analyst would ask: "what am I actually trying to understand?" The model has no stake in the analytical question — it just generates plausible-looking charts.

For a six-column weather dataset, plausible-looking means: a temperature trend, a precipitation bar chart, a wind speed summary. Not wrong, but not interesting. An analyst with domain knowledge would ask: does precipitation correlate with temperature range? Is there a seasonal pattern to the wind? Which weather type contributes most to total precipitation?

Those are hypotheses — statements about what might be true — and they produce more targeted, more insightful charts.

## Goal Exploration: commit to intent before planning charts

In the platform’s visualisation module (called `elt_llm_visualise` in this codebase), when `mode: infer_visualisations` is set, the pipeline runs a Goal Exploration step before any chart is planned.

The LLM receives a compact statistical summary of the dataset — column types, cardinalities, null rates, value distributions, date ranges. It does not receive raw rows.

It is then asked to generate 3-5 analytical goals and hypotheses: specific questions this dataset could answer, areas of variance worth investigating, and relationships between columns likely to surface insights. It must commit to these hypotheses explicitly before designing any charts.

This is the compact statistical summary the LLM sees for the Seattle weather dataset (1,461 rows, 6 columns, 2012-2015):

```
date: temporal — 1,461 unique values, range 2012-01-01 to 2015-12-31
precipitation: numeric — range 0.0-55.9 mm, mean 3.03, std 6.68
temp_max: numeric — range -1.6-35.6 °C, mean 16.4
temp_min: numeric — range -7.1-18.3 °C, mean 8.2
wind: numeric — range 0.4-9.1 m/s, mean 3.03
weather: categorical — 5 values: drizzle, sun, rain, fog, snow
```

[NEEDS_DATA: actual LLM-generated Goal Exploration output — the 3-5 hypotheses the model generated for the Seattle weather run on 2026-04-10]

From those hypotheses, the LLM then plans one chart per hypothesis: transformation spec (filter, group-by, aggregate, sort), chart type, Vega-Lite mark, insight statement, narrative, and recommended action. The hypotheses constrain the chart planning — you cannot draw a chart that does not address a stated hypothesis.

## The result: what the Seattle weather run produced

Running the pipeline against the Seattle weather dataset (public, 1,461 rows, 6 columns, 2012-2015) on 2026-04-10 produced five charts in 6 minutes 28 seconds, with 6 LLM calls and 0 failures:

- **Seattle temperature trends (2012-2015)** — temporal line chart
- **Seasonal weather composition** — normalised stacked bar by month
- **Precipitation vs temperature range** — scatter or relationship chart
- **Precipitation distribution** — histogram with distribution profile
- **Wind speed by weather type** — bar/box by weather category

Each chart addresses a distinct analytical question. Compare that to the "plausible but generic" set you would get from asking "what charts should I draw?" — which almost always returns the same three or four chart types regardless of dataset.

```yaml
inference:
  top_k: 5
  enable_self_critique: true
  llm:
    provider: "ollama"
    model: "qwen3.5:9b"
    temperature: 0.1
    max_tokens: 2500
  prompt: |
    Create a publication-quality analysis pack explaining seasonality,
    precipitation patterns, and temperature trends in Seattle weather.
```

`enable_self_critique: true` adds one additional LLM call after planning. The model reviews its own plan for chart quality, correct aggregation, and narrative usefulness before any chart is rendered. This is covered in detail in Part 4.

## Why this reduces hallucination

There is a secondary benefit to Goal Exploration that is not immediately obvious: it reduces column hallucination.

When the LLM plans charts directly, it occasionally invents column names — it "remembers" that datasets like this one often have a `season` column or a `month_name` derived field, and it proposes charts based on that memory rather than the actual schema. When those charts hit the validation layer, they fail with a column reference error.

When the LLM writes hypotheses first, it is forced to reason from the actual column list in the summary — `date`, `precipitation`, `temp_max`, `temp_min`, `wind`, `weather` — before making any column references. It cannot hallucinate a column that was not in the summary it just processed.

This is not a perfect defence. But it reduces the frequency of column reference failures, which directly reduces the number of LLM repair calls the pipeline needs to make.

## The cost

Goal Exploration adds one LLM call per run. For the Seattle weather run: 6 total LLM calls across goal exploration, chart planning (5 charts), and self-critique. At local Ollama inference rates, `cost_gbp_est: 0.0`. For cloud-hosted models, this would add one inference call per pipeline run — a manageable cost for the quality improvement.

The total wall time of 6m28s includes goal exploration, planning, self-critique, and Vega-Lite rendering for all five charts. Goal exploration adds approximately one LLM call's latency — roughly 30-60 seconds at local inference speeds.

[NEEDS_DATA: per-stage timing breakdown from a run with Goal Exploration enabled — goal exploration time vs planning time vs render time]

## The lesson

Making the LLM commit to analytical intent before producing output is not a UX feature. It is a reliability pattern.

A model that is forced to ask "what is worth investigating?" before "what should I draw?" produces charts that address questions worth asking. A model that skips straight to chart output produces charts that are plausible but undirected.

The same principle applies to any AI system that produces artefacts: require the model to state its intent before you accept its output. What the model says it is trying to do and what it produces should be verifiable against each other.

## Limits / when not to use

- Hypothesis-first workflows trade breadth for relevance; they can miss surprises that only broad exploration finds.
- If the input summary is poor, hypothesis quality collapses; the system needs a reliable data summariser.
- Treat timings as workload-dependent; model choice and validation gates change runtime.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; [NEEDS_DATA: dataset used (public), model settings affecting timing]
- Dataset class: public tabular dataset (publish-safe)
- Non-reproducible from this article: client-specific reporting packs and internal datasets

---
