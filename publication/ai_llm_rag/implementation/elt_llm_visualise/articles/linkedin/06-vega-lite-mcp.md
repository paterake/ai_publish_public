If charts are config, they can be tools.
MCP turns “generate a report” into a bounded, callable operation.
So I exposed the visualisation pipeline as an MCP tool surface.

When an AI generates a chart, you have two options: generate code and execute it, or generate a declarative spec and validate it.

I use Vega-Lite specs. Here is why the distinction matters:

- Vega-Lite has a published schema (v5). You can validate a spec before rendering — without running any code.
- A validated spec that fails is a structured error you can repair. A matplotlib exception is a stack trace you have to debug.
- Specs are diffable JSON. You can check whether a repair attempt actually fixed the problem, or whether it changed what the chart means.

The MCP (Model Context Protocol) pattern makes this composable. The assistant reasons in chat. When a chart is needed, it calls a rendering tool that accepts a Vega-Lite spec and returns a PNG. Two tools. One stable contract.

What makes production MCP charting work vs what makes demos work:

- Schema validation pinned to v5, with human-readable errors the assistant can act on
- Field reference validation — the spec references `precipitation_mm`; the dataset has `precipitation`. Catch it before the chart renders empty.
- A row cap on the data tool — 100K rows produces a slow render or a massive file

📌 None of those are optional. They are the difference between a tool the AI can use reliably and one it babysits.

Article linked below — includes the exact setup steps for a local Vega-Lite MCP server.

Are you using MCP tools in your AI workflows? What has been the hardest part to get right?

#DataVisualization #DataEngineering #AI #Python #LLMOps
