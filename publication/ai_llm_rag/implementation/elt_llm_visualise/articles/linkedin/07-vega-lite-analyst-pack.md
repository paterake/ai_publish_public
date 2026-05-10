Executives don’t want charts. They want a pack.
The pack must be repeatable, explainable, and fast to regenerate.
So the pipeline outputs a Vega-Lite analyst pack from config.

Getting a chart from an AI is easy. Getting five charts you can share with a stakeholder, with confidence in what they show, is a different problem.

This is the complete workflow I built:

1. You define the dataset and the question in YAML
2. The runtime summarises the dataset — no raw rows to the LLM
3. Goal Exploration: the LLM writes analytical hypotheses before proposing charts
4. Prompt guardrails constrain Vega-Lite patterns to what the validator accepts
5. Transformations execute deterministically in Pandas — group-by, aggregate, sort, top-N
6. Vega-Lite specs validate against v5 schema in strict mode before rendering
7. Failed specs trigger a single targeted repair — insight and transformation preserved, only the spec is corrected
8. Output: charts, combined report, audit manifest

Two key design decisions that make this production-grade:

📌 No exec(). The LLM returns JSON. The runtime executes it. A hallucinated column name fails at validation, not at a Pandas KeyError buried in a 400-line notebook.

Strict semantic boundary. The validator allows schema-safe fixes. It blocks semantic rewrites. If a chart cannot be rendered without changing what it means, it fails loudly. Silent wrong output is worse than a loud correct error.

Article linked below — includes full setup, config examples, and the rationale behind each design decision.

What is your standard for "good enough to share with a stakeholder"?

#DataVisualization #DataEngineering #AI #Python #LLMOps
