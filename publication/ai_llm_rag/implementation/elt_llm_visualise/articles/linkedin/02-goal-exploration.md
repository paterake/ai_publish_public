Most AI chart tools go straight to: “pick a chart”.
That produces plausible visuals with no analytical stake.
So I force hypotheses first, then let the pipeline plan charts.

Most AI visualisation tools ask the model one question: "what chart should I draw?"

The model returns a bar chart. It is probably correct. It is almost certainly generic.

The problem: the model went straight to output without doing any analytical work.

I built a step called Goal Exploration that runs before any chart is planned. The LLM receives a statistical summary of the dataset — no raw rows — and is asked to generate 3-5 analytical hypotheses. Questions the data could answer. Relationships worth investigating. It must commit to these hypotheses before it is allowed to propose a single chart.

Running this against the Seattle weather dataset (1,461 rows, 6 columns, 2012-2015) produced five charts in 6m28s on a local model:

- Temperature trends over 4 years
- Seasonal weather composition
- Precipitation vs temperature range
- Precipitation distribution
- Wind speed by weather type

📌 Not a generic "here's the data." Five specific analytical questions answered.

Secondary benefit: when the model writes hypotheses from a column list, it hallucinates column names less often. It reasons from the actual schema rather than from memory of similar datasets.

Article linked below — Part 2 of my series on local-first AI visualisation.

When you use AI for analysis, do you find it produces answers to the question you asked, or to a generalised version of it?

#DataVisualization #DataEngineering #AI #Python #LLMOps
