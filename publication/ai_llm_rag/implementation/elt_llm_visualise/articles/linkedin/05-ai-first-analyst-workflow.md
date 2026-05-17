The mistake is treating LLM visualisation as a chart generator.
The value is an analyst workflow: intent, evidence, critique, outputs.
So I built an AI-first loop that produces artefacts you can audit.

There is a version of "AI-assisted analysis" that does not change how you work. You write SQL, export a CSV, open a notebook, and at some point ask an AI to suggest a chart type. It says "bar chart." You were already going to use a bar chart.

The workflow I built works differently. You write the question. The pipeline answers it.

From a YAML config and a dataset, in one command:

- The runtime profiles the dataset (no raw rows to the LLM)
- Goal Exploration forces the LLM to write an analysis brief before touching a chart
- The LLM returns structured JSON — transformation logic, chart spec, insight, narrative, action
- Self-critique reviews the plan before rendering
- Vega-Lite specs are schema-validated; hallucinated column names fail before execution
- Output: per-chart HTML, a combined report, and a reproducible run manifest

The Seattle weather dataset (1,461 rows, 6 columns) produced five charts in 6m28s on a local model. Zero LLM failures. Zero external API calls. Cost: £0.

📌 The analyst's job changes from chart production to question definition. The pipeline handles the repeatable work.

Article linked below — Part 5 of my series on local-first AI visualisation.

What would you do with the time you currently spend on manual chart production?

#DataVisualization #DataEngineering #AI #Python #LLMOps
