# Config-Driven Charts Without Code

**Series:** LLM-Driven Data Visualisation — Part 1 of 7

---

Most reporting pipelines have one thing in common: nothing is recorded. No manifest, no reproducibility, no way to definitively explain last quarter’s charts. This is the architecture of a config-driven visualisation pipeline: YAML in, auditable report out, zero LLM calls required.

## Who this is for

You build recurring reports and dashboards and you want the “what to show” decisions to be explicit, versioned, and repeatable.

## What you’ll learn

- Why config-driven visualisation is an architecture pattern, not a convenience feature
- How “separate specification from execution” reduces maintenance churn as datasets change
- What makes an automated report auditable instead of “a notebook someone ran once”

## Constraints

- The output must be reproducible from a config file, not a notebook state
- The report must carry an audit trail of what was requested and what was generated

Most data reporting pipelines look like this: write a SQL query, export to CSV, open Excel or a Python notebook, decide what charts to draw, format them, export images, paste into a slide deck. Do it again next month.

The problem is not the tools. The problem is that every step requires a human decision, and none of those decisions are recorded anywhere. When the dataset changes or someone asks "what exactly did you show last quarter?", you start over.

In the platform’s visualisation module (called `elt_llm_visualise` in this codebase), explicit mode replaces the manual steps with a config file. The analyst describes what to show in YAML. The pipeline reads the dataset, executes transformations in Pandas, and writes an HTML report with an audit trail. No Python chart code. No decisions lost in a notebook.

## Why most reporting pipelines are not reproducible

Most practitioners who need to automate reporting reach for matplotlib or Plotly scripts. The pattern is familiar:

```python
df = pd.read_csv("data.csv")
fig = px.bar(df.groupby("region")["revenue"].sum().reset_index(), x="region", y="revenue")
fig.write_html("revenue_by_region.html")
```

This works for one chart. For a ten-chart dashboard, you have ten separate code paths, each making implicit assumptions about column types, aggregation order, and output location. When the dataset changes, you debug each chart separately. When a stakeholder asks to add a filter, you change code.

The deeper issue: the chart definition and the execution logic are mixed together. There is no declarative record of what was requested — only imperative code that does it.

## The decision: separate specification from execution

Explicit mode inverts this. You write what you want in YAML:

```yaml
mode: explicit

dataset:
  path: data/sales_data.csv
  format: csv

visualisations:
  - name: revenue_by_region
    type: bar
    columns:
      x: region
      y: revenue
    options:
      title: "Revenue by Region"

  - name: revenue_trend
    type: line
    columns:
      x: date
      y: revenue
    options:
      title: "Monthly Revenue Trend"

output:
  directory: <output_dir>
  write_manifest: true
```

The pipeline reads the dataset, validates that the referenced columns exist, applies any transformations, and renders the charts. Zero LLM calls. Zero Python chart code.

### What the runtime does and does not do

The runtime executes the specification deterministically:

- **Does:** validates column names against the actual dataset before rendering
- **Does:** applies group-by, aggregate, sort, and top-N transformations where specified
- **Does:** writes each chart as a standalone HTML file, plus `index.html` for navigation and `report.html` with narrative panels
- **Does not:** guess your intent — if a column does not exist, it fails loudly

That last point matters for reproducibility. Silent wrong output is harder to debug than a loud correct error.

## What you get: the output artefacts

A run produces four artefacts:

**`report.html`** — the shareable analyst pack. Each chart is accompanied by a narrative panel (when provided in config). Can be sent directly to stakeholders without any dependencies.

**`index.html`** — navigation index for browsing individual charts.

**`manifest.json`** — the audit trail. Records `run_id`, `config_file`, `mode`, dataset dimensions, per-chart output paths, and `total_duration_seconds`. An example from a real test run (2026-04-01, sample data, 4 columns):

```json
{
  "run_id": "2026_04_01_094127",
  "mode": "explicit",
  "dataset": {
    "format": "csv",
    "columns": 4
  },
  "visualisations": [
    {
      "name": "age_by_city",
      "type": "bar",
      "inferred": false
    }
  ],
  "llm_calls": 0,
  "total_duration_seconds": 0.28
}
```

`llm_calls: 0` and `total_duration_seconds: 0.28`. Explicit mode is fast and deterministic because there is no LLM involved.

## Running it

Run the pipeline in explicit mode with a YAML config and a public dataset. The runtime validates column references before rendering and writes a shareable `report.html` plus an auditable `manifest.json`.

## The practical value

The audit trail is what makes this useful beyond demos. `manifest.json` records exactly what ran, with what config, against what data, producing which outputs. Two engineers running the same config against the same dataset get identical outputs — and the manifest proves it.

The YAML config is also version-controllable. Tracking chart definitions in git means you can answer "what were we showing in the Q1 board pack?" without opening a notebook.

[NEEDS_DATA: screenshot of report.html from an explicit-mode run on a public dataset — showing at least 2 charts with the audit manifest visible]

## The limit of this approach

Explicit mode requires you to know which charts you want. For a familiar dataset with a well-defined reporting requirement, this is the right choice — fast, reproducible, no LLM cost.

But for an unfamiliar dataset, or when the question is "what does this data tell me?", you need something different. That is what the next article covers.

---

## Limits / when not to use

- If stakeholders need ad-hoc exploration, config-driven reports can feel restrictive; it is best for repeatable packs.
- Automated charting must be validated; without schema checks and manifests, failures become silent.
- For sensitive datasets, publish-safe examples must use public equivalents; do not embed client schema conventions in the article.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; explicit mode (0 LLM calls); manifests enabled
- Dataset class: public tabular dataset (publish-safe)
- Non-reproducible from this article: any internal reporting contracts or client-specific packs

---
