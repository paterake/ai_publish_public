# Two Query Surfaces, One Orchestration Layer

## Key takeaways

- One orchestration layer, two contract surfaces: MCP for developers and IDE assistants, HTTP for consumers and systems. The routing logic and tool behaviour are identical — only the integration contract differs.
- Not every question needs RAG. Route to the cheapest correct source: deterministic JSON lookup for extracted records, graph traversal for relationships, RAG only for unstructured text that requires synthesis.
- The decision rule for adding a new tool: "Can a junior dev script this directly from the file?" — if yes, don't use RAG. Applying this rule prevented days of wasted effort trying to RAG over structured JSON and XML sources.

---

Think of an AI assistant as a person walking into a library with a question. The librarian — the MCP server — does not store the information. They know where everything is, what each source contains, and how to retrieve it usefully. They hold the catalog. The books, the databases, the digital files: those are the information stores. The librarian decides what is *findable*. The content producers decide what exists. These are different responsibilities, and keeping them separate is the whole design.

The hard part of agents isn’t the models. It’s the contracts: what tools the librarian offers, what they return, and what’s allowed. This is the architecture for exposing one orchestrator via both MCP and HTTP, with strictly bounded tools.

It is deciding what the assistant is allowed to do, how it proves what it did, and how you expose the same capability to multiple clients without rewriting the system for each integration.

This article is the story of building two query surfaces on top of one knowledge base:

- an MCP server for developer tooling and IDE assistants
- an HTTP agent for non-technical consumers and systems

Both call the same orchestration layer. The difference is the contract surface.

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

## Why Integration Sprawl and Unbounded “Agent Power” fails at scale

Teams build a useful internal RAG system, then immediately hit a second problem: how do people actually use it?

The naive path is to integrate separately:

- a bespoke VS Code plugin
- a separate integration for a desktop assistant
- a separate API wrapper for programmatic callers

Each integration becomes its own little product. And the agent surface drifts: different defaults, different
tool sets, different “what is allowed” rules.

At the same time, most “agent” examples start from an unsafe premise: give the model tools, let it decide.
That works until the first time it runs the wrong command or returns a fluent answer with no evidence.

---

## What most teams try first: Ship One Interface and Keep Adding Exceptions

If you ship only a chat UI, engineers ask for programmatic access.
If you ship only an API, non-technical users ask for a UI.
If you ship an IDE integration, someone asks for it in a different IDE.

The first version is always simple. The second version is where you either create a contract surface, or you
end up with multiple forks of the same system.

---

## How the solution works: One Orchestrator, Two Contract Surfaces

The core design decision is that “asking the knowledge base” is not an interface. It is a capability.

So we treat the orchestration logic as the stable centre, and we expose it through two surfaces:

1. **MCP server** — tools, resources, and prompts (for IDE assistants and MCP clients)
2. **HTTP agent** — a tool-calling agent wrapped behind a web API (for consumers and systems)

The interfaces differ. The orchestration layer does not.

That prevents drift: the same tool implementations, the same defaults, the same quality posture.

---

## Query Routing: Not Every Question Needs RAG

The MCP layer routes each query to the cheapest data source that gives a correct, stable answer. This is not a
default assumption — it is the core design decision.

| Query type | Tool | Data source | Why not RAG? |
|------------|------|-------------|--------------|
| Entity definition or governance rules | `lookup_json_record` | Catalog JSON | Definition is already extracted and stored; deterministic, low-latency |
| Candidate / gap entity | `lookup_json_record` | Candidate catalog JSON | LLM already ran during the retrieval pipeline |
| Reference values and enumerations | `lookup_json_record` | Reference data JSON | Enumerated lists — no inference needed |
| Relationship traversal, governance chain | `query_graph` | Knowledge graph (NetworkX) | Graph traversal is O(hops), not O(chunks); no LLM variance |
| Governance rules, obligations, policy text | `query_rag` | ChromaDB + BM25 | Unstructured; requires semantic retrieval across many chunks |

**The decision rule for adding a new tool:**

> “Can a junior dev script this directly from the file?” — if yes, don’t use RAG.

This rule prevented days of wasted effort trying to RAG over enterprise architecture model XML and inventory JSON.
Both are structured files with stable keys — a direct read is faster, deterministic, and produces a better answer.
RAG adds LLM variance with no benefit when the file can be parsed directly.

**Compound queries — chaining tools for complex questions:**

Some questions span multiple data sources. The MCP host (Claude) chains tool calls:

```
“Tell me about Club governance”
  1. lookup_json_record(“Club”)        → formal definition from the catalog JSON
  2. query_graph(“Club”, hops=1)       → direct governance relationships
  3. query_rag(“Club governance rules”) → handbook obligations and rule citations
  → Synthesise all three into one answer
```

No single tool can answer this correctly. The value of the orchestration layer is that all three can be called
in sequence by the host and composed into a grounded answer.

## What Makes MCP Production Work (It’s Not the Protocol)

MCP is the wire format. Production work is everything around it:

- **Tool contracts**: input schema, output schema, error taxonomy
- **Allowlist posture**: only explicitly approved pipeline commands are runnable
- **Transport discipline**: stdio servers must keep stdout clean; only protocol messages are emitted
- **Config-driven defaults**: tool defaults and resource mappings are not hardcoded; they are declared

**Confidence scoring:** `query_rag` returns a `confidence_score` field alongside the answer and sources. It
averages the top retrieval scores and applies `1 - exp(-avg)` to map to a 0–1 range. Low confidence is a
signal to the host to flag the answer for human review rather than accepting it silently.

This is the part that looks boring in demos and is essential in real systems.

Here is the minimal shape of a “safe tool surface”:

- tools return structured JSON with explicit error fields
- long-running tools are bounded and observable
- resources resolve to files via templates (output roots, collection prefixes)
- pipeline execution is allowlisted (no arbitrary shell execution)

---

## Result: Two Interfaces, One Behaviour

When the contract surface is explicit, you can expose the same system to different clients without rewriting
the engine.

The MCP server becomes the developer interface: query, look up a record, list collections, run a pipeline.

The HTTP agent becomes the consumer interface: ask a question, receive an answer with sources and a trace of
what tools were called.

The tool contract shapes are stable. A `list_collections` call returns:

```json
{ "prefix": "governance_handbook", "collections": ["governance_handbook_s01", "governance_handbook_s02"], "count": 2 }
```

A `query_rag` call returns answer, sources with per-chunk scores, and a confidence score:

```json
{
  "answer": "…",
  "sources": [{ "metadata": { "collection": "governance_handbook_s07", "document_id": "…" }, "score": 0.82 }],
  "confidence_score": 0.87,
  "collections_queried": ["governance_handbook_s07", "governance_handbook_s40"]
}
```

A `query_graph` governance chain call returns a traversal result with no LLM involved:

```json
{ "query": "governance_chain", "entity": "Role", "chain": ["Role", "…", "the governing body"] }
```

---

## Architecture Decisions Summary

| Decision | Alternative Considered | Why This |
|----------|----------------------|---------|
| One orchestration layer, two surfaces (MCP + HTTP) | Separate implementations per surface | Single implementation prevents tool behaviour drifting between surfaces |
| Route to cheapest correct source (JSON → graph → RAG) | RAG for all questions | Deterministic sources eliminate LLM variance on questions that don't need synthesis |
| Decision rule: “can a junior dev script this from the file?” | Case-by-case judgment | Explicit rule prevented RAG from being applied to structured JSON and XML — saved implementation effort and degraded answers |
| Allowlist posture for pipeline execution | Open shell execution | Unbounded shell access from an MCP server is a security risk; allowlisting limits blast radius |
| Config-driven defaults (collection prefix, paths, iterative mode) | Hardcoded values in server code | Domain-agnostic server; reference config ships separately and is swapped per deployment |
| Confidence score on `query_rag` output | Answer only | Host needs a signal to trigger human review; confidence without a formula is noise |
| Structured error returns (never exceptions) | Let exceptions propagate to host | MCP hosts cannot handle unstructured exceptions; structured errors allow the host to display or route them gracefully |
| stdio transport as default; streamable-http as optional | HTTP-only | stdio is the correct transport for local Claude Desktop / IDE integrations; stdout cleanliness is mandatory |

---

## The Lesson: Treat “Agent” as a Contract Problem

If you think of an agent as “a model that can do things”, you will build an unsafe system.

If you think of an agent as “a caller of bounded tools with explicit contracts”, you end up with a system that
can be exposed widely without losing control.

MCP is valuable because it makes the contract surface portable across clients.
The orchestrator is valuable because it keeps behaviour consistent across surfaces.

That is the path from “demo agent” to “operator surface”.

## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; `defaults.iterative=true`; `defaults.collection_prefix=<prefix>`; `paths.rag_config=<rag profile>`; `paths.retriever_config=<retriever profile>`; JSON lookup schema `{list_key, name_key}`
- Dataset class: internal governance handbook + enterprise architecture model (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## Limits / when not to use

- Remote transports require authentication and rate limiting before they are production-safe.
- The allowlist posture must remain tight; a permissive tool surface is a security risk.
