Most reporting pipelines have one thing in common: nothing is recorded.
No manifest, no reproducibility, no way to explain last quarter’s charts.
So I made reporting config-driven: YAML in, auditable report out.

SQL written, CSV exported, notebook opened, chart formatted, image pasted into a slide. Do it again next month. No audit trail. No reproducibility. No way to answer "what exactly did you show last quarter?"

I built an explicit visualisation mode that replaces the manual steps with a YAML config. You describe what to show — chart type, columns, transformations. The pipeline executes it deterministically and writes:

- Per-chart HTML files
- A combined report with narrative panels
- A `manifest.json` audit trail: config fingerprint, dataset fingerprint, runtime, outputs

0 LLM calls. 0.28 seconds for a 4-column dataset. Identical output every run.

📌 The YAML config is version-controlled. "What were we showing in the Q1 board pack?" becomes a git log query.

This is Part 1 of my series on local-first AI visualisation. Article linked below.

What part of your current reporting workflow would you most want to make reproducible?

#DataVisualization #DataEngineering #AI #Python #LLMOps
