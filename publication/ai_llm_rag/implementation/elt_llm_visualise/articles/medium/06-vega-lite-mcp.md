# Vega-Lite via MCP: AI-Specified Charts Without Code Generation

**Series:** LLM-Driven Data Visualisation — Part 6 of 7

---

If charts are config, they can be tools. Model Context Protocol (MCP) turns “generate a report” into a bounded, callable operation. This is how to expose a local visualisation pipeline as an MCP tool surface.

## Key takeaways

- MCP is an interface contract: tools are bounded operations with schemas, not free-form code execution.
- Vega-Lite works because specs are schema-validatable before rendering.
- Production reliability requires hard gates: field validation, schema errors you can act on, and row caps to prevent unbounded output.

## Who this is for

You are building or operating an AI-enabled data workflow and you care about reliability, auditability, and repeatability more than demos.

## What you’ll learn

- The key design decisions that made the final implementation work
- The turning points (attempt → failure → decision → proof) that justify the architecture
- The limits and where the approach does not apply

## Constraints

- Local-first / zero data egress where relevant
- Outputs must be auditable (sources, artifacts, manifests)

Most AI assistants can describe a dataset in detail. The bottleneck is turning that description into a chart a human can review, share, and iterate on — without writing plotting code.

The Vega-Lite Model Context Protocol (MCP) pattern closes that gap. The assistant reasons in chat. When a chart is needed, it calls a rendering tool that accepts a declarative Vega-Lite spec and returns a rendered output. No Python. No `exec()`. No chart library API calls embedded in conversation.

This article explains why Vega-Lite is a good fit for this pattern, what the MCP contract looks like in practice, and the exact setup you can hand to a coding assistant to get a working local installation.

## Why Vega-Lite and not Plotly, matplotlib, or D3

The practical reason is validation. Vega-Lite charts are declarative JSON objects with a published schema (v5). Every field has a defined type and a valid value set. A validator can check a Vega-Lite spec for correctness before rendering — without running any code.

Compare that to Plotly or matplotlib: both require generating Python function calls. You cannot meaningfully validate "does this `px.bar()` call produce a correct stacked normalised bar chart?" without executing it. Schema validation is not available because there is no formal schema for imperative chart code.

The second reason is diffability. A Vega-Lite spec is a JSON object you can store, version, diff, and reason about. You can check whether two specs are equivalent, whether a spec change affects the chart type or the data encoding, and whether the model's repair attempt actually fixed the problem. Imperative plotting code does not offer this.

The third reason is portability. A Vega-Lite spec renders identically in any environment with a Vega-Lite renderer. The same spec that renders in a local HTML file renders in Observable, in a Jupyter notebook with `vega_datasets`, or in any MCP client that supports Vega-Lite output. The rendering environment does not affect the spec.

## The trust boundary this creates

When an assistant generates a Vega-Lite spec and passes it to an MCP rendering tool, the trust boundary is clear:

- **The assistant proposes:** chart type, encoding fields, transformation hints, colour scheme, title
- **The MCP tool proves:** whether the spec is schema-valid, whether the field references match the loaded dataset, whether the rendered output is correct

The assistant cannot produce a chart that bypasses this validation. If the spec is invalid, the tool returns a schema error — structured, human-readable, and actionable. The assistant can read the error and repair the spec. This is a deterministic repair loop, not a silent failure.

## The two-tool MCP contract

Most Vega-Lite MCP servers expose two operations, and this simplicity is intentional:

**`save_data`** — persist a dataset under a name
- Input: dataset name + rows (array of JSON objects)
- Effect: the server stores the dataset for subsequent renders

**`visualize_data`** (or `visualise_data`) — render a Vega-Lite spec
- Input: dataset name + Vega-Lite JSON spec (as string or object)
- Output: rendered artefact (PNG as base64, or Vega-Lite spec text for verification)

Two tools. One contract. The simplicity reduces tool confusion — the assistant always knows which tool to use and what to pass.

## Local installation (conceptual)

You run a local MCP tool server that implements the Vega-Lite contract, then register that server with your assistant so the assistant can call `save_data` and `visualize_data`. Keep the server local to avoid data egress. The exact registration steps vary by client; the important part is the contract shape, not the specific implementation.

### A first test prompt

Once the tools are available, ask the assistant to:

1. Save a small dataset (5-20 rows) under a name
2. Render a simple bar chart spec against it

Example dataset shape to give the assistant:

```json
[
  {"month": "Jan", "precipitation_mm": 147},
  {"month": "Feb", "precipitation_mm": 99},
  {"month": "Mar", "precipitation_mm": 97},
  {"month": "Apr", "precipitation_mm": 43}
]
```

Expected Vega-Lite spec the assistant should produce:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "mark": "bar",
  "encoding": {
    "x": {"field": "month", "type": "nominal"},
    "y": {"field": "precipitation_mm", "type": "quantitative"}
  }
}
```

If the MCP server is running and the assistant calls both tools correctly, you should receive a PNG of a bar chart.

## What the happy path misses

The most common operational problems in production Vega-Lite MCP workflows are not about chart aesthetics. They are about reliability:

**Near-JSON from the assistant** — trailing commas, unescaped quotes, comments. The assistant generates something that looks like JSON but is not. A validator that can parse and repair common JSON syntax errors is worth having.

**Near-valid Vega-Lite** — wrong enum values (`"type": "cat"` instead of `"type": "nominal"`), wrong nesting (`{"stack": {"value": "normalise"}}` instead of `"normalise"`), missing required fields. Schema validation pinned to v5 catches these before the renderer runs.

**Field name mismatch** — the assistant references `precipitation` but the dataset has `precipitation_mm`. This is caught at render time if the tool validates field references against the loaded dataset. If it does not, the chart renders empty silently.

**Unbounded output** — a 100,000-row dataset produces a massive PNG (or a slow render). A row cap on the `save_data` tool prevents this.

When choosing or building an MCP server, insist on schema validation (with human-readable errors), field reference validation, and a row cap. These are not optional hardening — they are the requirements that distinguish a reliable tool from one that fails silently.

## Where this pattern is most useful

**Interactive exploration** — ask a question, see a chart, refine the question. The round-trip of "spec → render → review → revise spec" is fast enough to iterate in conversation.

**Reproducible artefacts** — specs can be saved alongside conversation history. Two people following the same conversation can reproduce the same charts.

**Multi-assistant workflows** — different assistants can share the same rendering tool. The spec format is the stable interface; the rendering environment is a detail.

## Limits / when not to use

- MCP tool surfaces must remain tightly allowlisted; permissive tool execution is a security risk.
- Tool contracts need stable schemas; frequent breaking changes will erode trust in automation.
- Remote transports require authentication and rate limiting before production use.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; MCP tool surface is stdio; Vega-Lite renderer `strict=true`; report outputs are HTML with embedded index; tool allowlist enforced
- Dataset class: Iris (public Vega dataset; JSON)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---
