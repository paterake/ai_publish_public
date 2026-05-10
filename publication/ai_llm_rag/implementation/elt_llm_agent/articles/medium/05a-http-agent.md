# The HTTP Knowledge Base Agent (Audit Trail First)

A fluent answer without an audit trail is a demo. Users need the tool calls and the evidence, not just the response. This is how to build an HTTP agent that returns traces and sources alongside every answer.

## Key takeaways

- An agent surface without tool tracing is a demo. Return the ordered tool call list and sources as first-class fields alongside every answer.
- Route to the cheapest correct source first: deterministic catalog lookups and graph traversal before RAG, because RAG adds LLM variance with no benefit when the answer is already extracted.
- A RAG tool call may itself invoke an LLM for synthesis — combined with agent synthesis, one user question can produce multiple LLM calls. Track this as an explicit cost, not a surprise.

The first naive implementation is also the fastest way to destroy trust: a fluent answer, no evidence, no way
to tell what the system actually did, and no way to diagnose why it failed.

This article is the story of building an audit-first agent surface over a multi-source knowledge base:

- deterministic catalogs and record lookups
- graph traversal queries
- RAG queries with explicit sources

All routed through bounded tools, and returned with a trace of what happened.

---

## Who this is for

You are building or operating an AI-enabled data workflow and you care about reliability, auditability, and repeatability more than demos.

## What you’ll learn

- The key design decisions that made the final implementation work
- The turning points (attempt → failure → decision → proof) that justify the architecture
- The limits and where the approach does not apply

## Constraints

- Local-first / zero data egress where relevant
- Outputs must be auditable (sources, artifacts, manifests)

## Why a Useful Agent Must Be Debuggable

If you can’t answer these questions, you don’t have a usable system:

- which tool was called, in what order?
- what evidence was used?
- what did the system do when evidence was missing?
- how do you reproduce the result?

Most “agent demos” don’t even try to answer them. That’s fine for a demo. It is not fine for a real system.

---

## What most teams try first: One Big Prompt and a Hope

The naive design is:

1. retrieve some chunks
2. ask the model to answer
3. print the answer

It fails in two predictable ways:

- it guesses when retrieval is incomplete
- it becomes impossible to debug when the user challenges the answer

Even adding citations doesn’t solve this if you can’t show the decision path the system took.

---

## How the solution works: A Tool-Calling Router with an Audit Trail

Instead of treating the model as the system, treat it as a router over bounded tools:

| Source type | Tool | Best for |
|-------------|------|----------|
| `json` | `lookup_entity` | Deterministic entity records — definition already extracted; low latency, zero LLM variance |
| `rag` | `query_knowledge_base` | Unstructured governance or policy text; requires semantic retrieval and synthesis |
| `graph` | `query_graph` | Relationship chains, neighbours, governance paths — O(hops), not O(chunks) |

The routing model matters for cost. A governance chain query goes directly to the graph — no synthesis call, result in milliseconds. A catalog lookup reads a JSON record — no embedding, no model. RAG is the right tool when the question requires evidence synthesis across multiple chunks. Everything else should be answered without it.

**One user question can trigger multiple LLM calls.** The RAG tool invokes the retrieval + synthesis pipeline — that is one LLM call. The agent then synthesises the final response from the tool output — that is a second. On a busy question this is expected behaviour, but it must be tracked explicitly as a cost, not discovered as a surprise in telemetry. The architecture documents this as an optimisation target: a retrieve-context-only path that skips agent synthesis for direct record lookups.

The output is not only “an answer”. It is:

- the ordered tool call list
- the sources returned by those tools
- the final response grounded in those sources

This is what makes the interface safe for non-technical consumers: the system can explain itself.

---

## Result: A Consumer-Facing Surface Over the Same Engine

With the orchestration layer stable, the HTTP agent becomes one surface among several. It is not a separate
system. It is a different contract for a different audience.

Proof points to capture before publishing:
- Publish-safe response envelope (contract-level shape):

```json
{
  "answer": "…",
  "tool_calls": ["lookup_json_record", "query_rag"],
  "sources": [
    { "tool": "lookup_json_record", "record": { "entity_name": "Role", "formal_definition": "…" } },
    { "tool": "query_rag", "metadata": { "collection": "governance_handbook_s07", "document_id": "…" }, "score": 0.82 }
  ]
}
```
- [NEEDS_DATA: measured latency/concurrency result with run conditions]

## Architecture Decisions Summary

| Decision | Alternative Considered | Why This |
|----------|----------------------|---------|
| Tool-calling agent with explicit routing | Single-prompt synthesis over all sources | Routing makes tool calls traceable; single prompt cannot attribute which source drove the answer |
| Tool output returned as first-class API fields | Return only the final answer | Consumers and downstream systems need the audit trail; answer-only is a demo contract |
| JSON and graph sources routed before RAG | RAG for all questions | Deterministic sources have zero LLM variance and millisecond latency — no benefit in RAG for structured records |
| Bounded tool set (allowlisted) | Open tool surface | Prevents the agent from taking unintended actions; essential for consumer-facing deployment |
| run_id per request, propagated through tool calls | Per-module logging only | Cross-tool correlation requires a shared identifier — without it, a multi-tool trace is unjoined fragments |
| Explicit "not found" outcome | Force an answer | Forcing an answer on missing evidence is a trust failure; the system must surface gaps as gaps |

---

## Limits / when not to use

- An agent surface without authentication, rate limiting, and auditing is not production-safe.
- The system must support “not found” outcomes; forcing an answer is a trust failure.
- Tool routers can still drift without contract discipline; the interface must remain bounded and debuggable.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; catalog cards `{json, rag, graph}` enabled; sources returned as first-class output; local inference
- Dataset class: internal governance handbook + enterprise architecture model (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## The Lesson

An agent is not a “smart answer machine”.

It is a bounded operator surface:

- contracts first
- evidence first
- audit trail always

That’s how you build something people can trust, and how you keep the system debuggable when it inevitably
hits edge cases.
