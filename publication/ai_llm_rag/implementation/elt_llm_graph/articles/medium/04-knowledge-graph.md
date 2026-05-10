# Building a Knowledge Graph from Unstructured Governance Documents

Here is what a failed relationship query looks like in a RAG-only system:

> "This is an important concept in enterprise governance. It relates to the organisation's operational framework and compliance requirements."

Technically fluent. Zero relationship content. No chain. No edges. If you need “what governs what?”, this failure mode is structural.

RAG is bad at relationships. If the analytical question is a path rather than a fact, synthesis is the wrong substrate. This is how to assemble a typed knowledge graph from audited catalogs to answer relationship queries deterministically.

If your question is “what governs what?”, “what is upstream of X?”, or “show me the chain from A to B”, a
synthesis model is doing the wrong job. You need traversal semantics, not more prompt engineering.

This article explains how a typed knowledge graph sits on top of structured catalogs, and why it becomes the
most reliable way to answer relationship questions once you have crossed a certain scale.

## Key takeaways

- RAG answers "what is X?" — it cannot reliably answer "what governs X?" because the answer is a graph shape, not a paragraph.
- A typed graph assembled deterministically from audited catalogs gives you traversal queries in milliseconds with zero LLM variance.
- The LLM's role changes: it explains what the graph returned, rather than generating the relationships itself.

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

## Why relationship questions don’t fit search

The naive approach is to treat relationship questions as normal queries:

1. retrieve the most relevant chunks
2. ask an LLM to assemble an answer
3. hope the sources cover the full chain

That works for “what is X?” It fails for “how does X relate to Y?” because the answer is not a paragraph in any
single place. It is a graph shape across multiple sources.

Two things go wrong:

- **Lost edges:** the relationship is present, but split across sources and never retrieved together
- **Invented edges:** the LLM “fills in” a plausible chain when the evidence is incomplete

If the question is inherently relational, the solution is not “retrieve harder”. It is to represent the
relationships explicitly.

---

## What most teams try first: Keep Pushing RAG Harder

When teams first notice relational queries failing, they usually try one of:

- increase `top_k` and hope the missing edge appears
- add a reranker and hope the chain becomes coherent
- add follow-up prompting (“check your answer”, “cite sources”)

These help at the margins. They do not change the underlying problem: you are asking a synthesis model to do
graph traversal implicitly.

The turning point is realising that relationship queries should be answered by a different substrate entirely.

---

## How the solution works: A Typed Graph Built from Deterministic Artefacts

The platform already produces structured outputs:

- a catalog of entities, with fields and relationships
- a catalog of candidate terms and gaps
- reference lists that behave like deterministic lookup tables

Those are not “just outputs”. They are stable, auditable inputs for a graph build step.

So the design decision is:

1. Treat catalogs as the source of truth
2. Build a directed graph where nodes are typed (entity / reference_value / stub)
3. Preserve relationship edge types from the catalogs
4. Export the graph in formats that can be queried and visualised

This is deliberately not “AI-driven graph extraction”. AI has already done its work upstream in controlled,
quality-gated steps. The graph is deterministic assembly.

---

## What the Graph Looks Like

At runtime, the graph build step loads three structured inputs and constructs a directed graph:

```
Catalogs  →  Graph Loader  →  NetworkX DiGraph  →  Exports
                              (typed nodes)        (GraphML / Cypher / D3 JSON)
```

**Why typed nodes matter:**

- **entity** nodes are the stable things you want to query — ~625 nodes sourced from entity catalogs
- **reference_value** nodes capture controlled vocabularies and enumerations — 53 nodes from a dedicated reference data catalog
- **stub** nodes preserve graph connectivity when a relationship mentions a name that is not present elsewhere — ~216 nodes

Stub nodes deserve specific explanation. When the extraction pipeline finds "A governs B" in handbook text, both A and B become nodes. If B is not in any catalog — perhaps a term referenced in governance rules but not formally catalogued — it is added as a lightweight stub. The edge is preserved. The graph stays connected. Without stubs, those edges would be silently dropped and the graph would have holes that make governance chain queries unreliable.

Without those types, a graph quickly becomes a bag of strings. With them, you can filter, traverse, and export with predictable semantics.

**Export formats:**

| Format | File | Use |
|--------|------|-----|
| GraphML | `knowledge_graph.graphml` | Graph exchange standard; input to all query functions |
| Cypher | `knowledge_graph.cypher` | Neo4j / Kuzu import for persistent graph databases |
| D3 JSON | `knowledge_graph_d3.json` | Browser force-directed visualisation |

The graph is built once per catalog version and reused. All query surfaces — MCP tools, agent tools, ad hoc scripts — load the same GraphML artefact.

---

## Result: Traversal Becomes a First-Class Query

Once the graph exists, relationship questions become deterministic operations:

| Query | Method | What it returns |
|-------|--------|----------------|
| “What governs X?” | `get_governance_chain(“X”)` | Directed chain of `governed_by` edges to root — stops on cycles |
| “What is connected to X?” | `get_neighbours(“X”, hops=2)` | BFS up to N hops; optional `rel_type` filter |
| “How is A related to B?” | `get_all_paths(“A”, “B”)` | All simple paths up to depth 6, capped at 10 |
| “What entities are in domain D?” | `find_entities_in_domain(“D”)` | Filter nodes by domain attribute |
| “What are all entities of type T?” | `find_by_category(“T”)` | Filter by `entity_category` (role, process, document…) |

All lookups are case-insensitive. Results are milliseconds — no LLM call, no embedding, no synthesis.

This changes the product surface. You are no longer asking a model to “explain relationships”. You are asking
a graph to return relationships, then optionally asking a model to explain what the graph returned.

**Example run scale:** 894 nodes, 772 edges (run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; corpus: internal governance handbook + enterprise architecture model).

---

## Architecture Decisions Summary

| Decision | Alternative Considered | Why This |
|----------|----------------------|---------|
| Deterministic graph assembly from audited catalogs | LLM-driven graph extraction at query time | AI already ran upstream in quality-gated steps; graph is the output of that work, not a new inference pass |
| Typed nodes (entity / reference_value / stub) | Untyped bag of strings | Types enable filtering, traversal semantics, and predictable query results |
| Stub nodes for missing entities | Drop unresolvable relationship endpoints | Dropping edges silently breaks governance chains; stubs preserve connectivity and make gaps visible |
| NetworkX in-memory DiGraph | Persistent graph database (Neo4j) | Sufficient for thousands of nodes; no infrastructure dependency; GraphML serialises to disk for replay |
| GraphML as the canonical export | Cypher or JSON as primary | GraphML is the stable interchange format; Cypher/D3 are derived exports for specific consumers |
| Config-driven edge type filter | Hardcoded relationship set | Allows whitelisting specific edge types without touching code |
| Config-driven exclusion of low-confidence nodes | Include all | Keeps the traversable graph clean; `LIKELY_FALSE_POSITIVE` stubs don't pollute governance chains |

---

## What Generalises (The Lesson)

The transferable lesson is simple:

- Use RAG when the answer is text.
- Use graphs when the answer is relationships.

The mistake is trying to make one substrate do both jobs.

If you already have structured outputs (catalogs, extracted entities, relationship fields), the fastest path to
relational reliability is to assemble a graph deterministically. The LLM becomes an explainer of what the graph
returned, not the generator of the relationships themselves.

## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; deterministic graph assembly from audited catalogs; exports GraphML / Cypher / D3 JSON
- Dataset class: internal governance handbook + enterprise architecture model (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## Limits / when not to use

- The graph is only as complete as the upstream extraction. Missing relationships upstream mean missing edges.
- An in-memory graph is excellent at thousands of nodes. At larger scales, you likely want a dedicated graph
  store, but the pattern stays the same: deterministic assembly from audited inputs.
