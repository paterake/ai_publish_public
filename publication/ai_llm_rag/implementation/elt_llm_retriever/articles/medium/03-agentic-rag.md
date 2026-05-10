# Quality-Gated Agentic RAG: When the First Pass Isn't Good Enough

Single-pass RAG loves boilerplate. When fetching sparse entities, it often replaces usefulness with fluency. This is how to fix it by introducing a strict quality gate and an agentic fallback loop that only triggers when the deterministic paths fail.

## Key takeaways

- Single-pass RAG fails predictably on sparse entities: fluent boilerplate is not evidence.
- A quality gate turns “looks OK” into a measurable decision: pass, escalate, or return “not found”.
- A bounded agentic fallback loop is cheaper than reviewing everything: escalation only happens when the first pass fails.

---

## Who this is for

You have a RAG pipeline that works for “common” concepts but collapses into boilerplate for sparse entities, and you need a programmatic way to gate quality.

## What you’ll learn

- How to score an LLM output for “usefulness” without pretending you can measure truth directly
- Why a bounded agentic loop is cheaper than reviewing everything by hand
- How the turning points (boilerplate failure, missing evidence) shaped the final design

## Constraints

- Outputs must be auditable (sources and a “not found” outcome are valid results)
- Cost must be proportional to difficulty (easy cases stay cheap; hard cases escalate)

## Why single-pass RAG fails for sparse entities

Standard RAG works like this: embed the query, retrieve the top K chunks, synthesise an answer. One retrieval call, one LLM call.

For a well-documented entity — a common concept with multiple handbook sections referencing it — this works well. For a sparse entity — a term defined briefly in one clause, or a category with no dedicated section — it produces boilerplate:

> "This is an important concept in enterprise governance. It relates to the organisation's operational framework and compliance requirements."

Technically fluent. Zero information content. Indistinguishable from a hallucinated non-answer.

The goal was to enrich 175 enterprise architecture model entities with structured definitions, governance rules, and relationships from an internal governance handbook. A single-pass approach would return generic answers for roughly half of them. That is not good enough for a knowledge base intended to drive downstream systems.

The solution: add a quality gate after every synthesis, and trigger a multi-step search loop when the gate rejects the output.

---

## How the pipeline works

The full enrichment pipeline has seven stages, but the core logic lives in three enrichment paths that differ by cost and quality:

```
Query → Retrieval → Synthesis → Quality Gate
                                    │
                        ┌───────────┼───────────┐
                     PASS         FAIL          FAIL (repeated)
                        │           │               │
                   cached_section  fast path    agentic loop
                   (0 RAG calls)  (1 RAG call)  (2-5 RAG calls)
```

Each path escalates LLM cost in exchange for better answers.

---

## Path 1: Cached Section (Zero LLM Calls)

During the discovery stage (Stage 1), the pipeline scans the entire handbook and identifies which sections contain each entity's name. When a section contains a dedicated passage for an entity, that passage text is cached directly.

At enrichment time, if a cached section exists, it is passed directly to the LLM for synthesis — no RAG retrieval at all. The text is already authoritative. There is nothing to retrieve.

For the handbook term catalog (431 candidates), approximately **95% take the cached section path**. The handbook is a well-structured glossary at its core, and most candidates have a dedicated definitional passage.

For the entity catalog (175 enterprise architecture model entities), the proportion is lower — model abstractions (Application, Interface, Data Object) are not explicitly defined in the handbook. Those entities need retrieval.

---

## Path 2: Fast Path (One RAG Call)

If no cached section exists, the pipeline runs a standard retrieval: embed the entity name, retrieve from the hybrid index (BM25 + dense vector, as described in the previous article), synthesise.

Then the quality gate runs.

### The Quality Gate

The gate scores the synthesis output on four weighted criteria:

| Criterion | Weight | What It Checks |
|-----------|--------|----------------|
| Length adequacy | 0.30 | `formal_definition` ≥ 200 chars, `governance_rules` ≥ 200 chars |
| Citation presence | 0.30 | At least one handbook section referenced |
| Non-generic | 0.20 | Does not contain boilerplate phrases ("important concept", "organisational framework") |
| Non-boilerplate | 0.20 | Specific, actionable content — not filler |

If the weighted score ≥ 0.5, the entity passes. The enriched record is written to the output catalog.

If the score < 0.5, the entity goes to the agentic loop.

The 0.5 threshold was chosen empirically. Lower values allowed too many generic answers through. Higher values sent too many borderline-but-acceptable answers into unnecessary LLM calls.

---

## Path 3: The Agentic Loop (Two to Five LLM Calls)

The agentic retriever runs a controlled search loop. At each iteration, the LLM receives:
- The current synthesis output (what it has so far)
- The current quality score
- A set of available actions

It selects one action:

| Action | What It Does |
|--------|-------------|
| `RETRIEVE` | Re-run the retrieval with a modified query |
| `KEYWORD` | Scan all section headings for the entity name — targeted lookup |
| `DONE` | Accept the current synthesis, stop iterating |

The loop runs until the LLM selects `DONE`, or the quality gate passes, or a maximum iteration limit is reached (5 iterations).

### Why KEYWORD Matters

The `KEYWORD` action is not a fallback — it is an intentional second search strategy. Dense vector + BM25 retrieval works from semantic similarity. For a rare or very specifically named entity, the query embedding may not pull the right chunks because the entity name appears in a section heading but not in the body text of any top-ranked chunk.

`KEYWORD` bypasses the embedding entirely and scans section headings for the exact entity name. This catches entries that vector search misses because the entity is named as a heading, not a body concept.

### The DONE Override

There is one edge case where the LLM can select DONE even when the quality gate has not passed: if no useful content was found and no keyword scan has been attempted yet, the loop would waste iterations on retrieval calls that will return the same results. The LLM is instructed to select KEYWORD in this case.

If the keyword scan also returns nothing, DONE is appropriate — the entity is genuinely absent from the handbook. The record is written with a low-confidence flag and moves to the manual review queue.

### A Concrete Example: Market Segment

"Market Segment" is an enterprise architecture model entity. It represents a classification concept in the organisation’s data architecture. The handbook does not have a section titled "Market Segment".

**Iteration 1 — RETRIEVE:** The vector query returns sections about commercial partnerships and broadcast segments. Relevant for background context, but no formal definition. Quality score: 0.31.

**Iteration 2 — KEYWORD:** Scans section headings. Finds a clause in the commercial regulations section where "market segment" appears in the context of broadcast rights classification.

**Iteration 3 — RETRIEVE (targeted):** Re-runs retrieval with "broadcast market segment classification rights" as the query. Returns the specific clause and three related clauses about segment definitions.

Quality score after synthesis: 0.67. Gate passes. Three iterations total.

This pattern — vector gives you the topic neighbourhood, keyword gives you the exact anchor, targeted retrieval gives you the context — repeats across most agentic fallback cases.

---

## What the system produced

### Entity Catalog (175 Enterprise Architecture Model Entities)

| Metric | Value |
|--------|-------|
| Total entities | 175 |
| BOTH (definition + rules) | 169 / 175 |
| Fast path | 156 entities |
| Agentic fallback | 17 entities |
| Average iterations (agentic) | 1.1 |
| Requires manual review | ~6 |

169 out of 175 entities have both a formal definition and governance rules. The remaining 6 are model abstractions that genuinely do not appear in the handbook — correctly flagged for manual review rather than silently filled with boilerplate.

### Handbook Catalog (431 Candidates)

| Metric | Value |
|--------|-------|
| Total candidates | 431 |
| Cached section path | ~95% |
| Fast path | ~4% |
| Agentic fallback | ~1% |
| Manual review queue | ~20 |

The handbook is the source document, so most entries have explicit passages. The agentic loop is rarely needed here — when it triggers, it usually means the candidate is a cross-reference term defined elsewhere in the handbook.

---

## The Two-Catalog Architecture

One design decision worth explaining: why two separate output catalogs rather than one consolidated one?

The intuition for consolidation is appealing: merge `entity_catalog.json` (enterprise architecture model entities) and `handbook_catalog.json` (handbook candidates) into one unified record per concept.

The problem: the two catalogs operate at different levels of abstraction.

- **entity_catalog.json** reflects the conceptual model — abstract entities like "Application", "Interface", "Data Object", "Market Segment". These are architectural abstractions that may or may not have operational definitions in the handbook.
- **handbook_catalog.json** reflects the handbook's own taxonomy — operational terms defined by the document rather than by the external model.

A consolidated merge would over-report: some conceptual-model entities have no handbook definition (correct — the handbook does not attempt to define them). Some handbook terms have no conceptual-model counterpart (correct — the handbook defines roles and rules that predate the model).

The 138 out of 175 "Not defined" entries in `entity_catalog.json` are the correct finding. The handbook operates at a more atomic, operational level and deliberately does not address all model abstractions. A consolidation pipeline would either generate false positives (linking handbook terms to model entities on weak matches) or mask the correct finding that the model has genuine definition gaps.

Two catalogs, with known and intentional overlap where it exists, is the honest representation.

---

## Architecture Decisions Summary

| Decision | Alternative Considered | Why This |
|----------|----------------------|---------|
| Quality gate after synthesis | Trust the LLM, no gate | ~50% of naive single-pass outputs were boilerplate |
| Four-criterion weighted gate | Single criterion (length) | Length alone misses fluent-but-generic outputs |
| Agentic loop with LLM action selection | Fixed retry strategy | Entity-specific context drives better search strategy |
| KEYWORD action as explicit step | BM25 handles this | Vector+BM25 still misses exact heading matches for rare terms |
| Two separate catalogs | Consolidated merge | Different abstraction levels; merge would mask definition gaps |
| Manual review queue for low-confidence | Exclude or hallucinate | Honest uncertainty tracking is more useful than false completeness |

---

## What This Produces

The output of this pipeline is a structured knowledge base: two catalogs with 606 total entries, each containing a formal definition, governance rules, handbook citations, and confidence scores.

This is not a chatbot that answers questions from a document. It is a structured knowledge artifact that can drive downstream systems — a governance glossary export (606 terms), a knowledge graph (894 nodes, 772 edges), and a query layer that any application can call.

The quality gate ensures that downstream consumers receive either a real answer with a cited source, or an explicit "not found" flag. No silent boilerplate. No unattributed hallucinations.

---

## Limits / when not to use

- A quality gate scores “usefulness”, not truth. It is a control mechanism, not a correctness proof.
- Agentic fallback must be bounded (iterations, budgets). Unbounded loops are a reliability and cost risk.
- Thresholds and scoring weights are domain-dependent; treat the numbers as measured for one corpus.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; quality gate `min_score=0.5`, `min_length=150`; agentic fallback `max_iterations=5`; local inference
- Dataset class: internal governance handbook + enterprise architecture model (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompt templates, internal contracts, and vendor-specific export wiring

---

*This is the third in a series on building a local-first AI platform for enterprise governance knowledge. The previous article covered hybrid retrieval (BM25 + dense vector + reranker). The next covers the knowledge graph built on top of these catalog outputs.*
