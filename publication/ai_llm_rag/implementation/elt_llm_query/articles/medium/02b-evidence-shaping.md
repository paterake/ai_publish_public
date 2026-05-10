# Evidence Shaping: Why “Top‑K” Is Not a Retrieval Strategy

Top‑k is not a retrieval strategy. More context often means worse answers. This is how to shape evidence — fusing, reranking, deduplicating, and ordering — before the LLM ever sees the context window.

They increase `top_k`.

Sometimes that helps. Often it makes answers worse: more context, less coherence, and a higher chance the model
locks onto the wrong chunk.

This article is the story of treating retrieval as an engineering pipeline, not a single parameter.

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

## Why Retrieval Isn’t Just Finding Something Relevant fails at scale

A good answer needs evidence that is:

- relevant
- diverse (not five near-duplicates)
- ordered (so the model sees the important parts in time)
- bounded (so you don’t drown the model in noise)

If you only optimise “relevance”, you get brittle behaviour. If you shape evidence deliberately, you get a
system that behaves predictably under change.

---

## What most teams try first: Keep Raising Top‑K

Raising `top_k` is appealing because it feels safe: “we won’t miss anything”.

But it creates two failure modes:

- **Noise inflation:** the context window fills with mildly related chunks that dilute the signal
- **Ordering failures:** the important chunk is present, but buried where the model never uses it

The turning point is recognising that retrieval needs the same discipline as any data pipeline: staged
processing with explicit decisions and guardrails.

---

## How the solution works: A Retrieval Pipeline with Explicit Stages

A reliable query layer tends to converge on stages like:

1. retrieve candidates from complementary retrievers (sparse + dense)
2. fuse results (not “pick one”)
3. rerank with a more expensive model when it pays for itself
4. shape context for coherence (diversity, deduplication, ordering)
5. synthesise with explicit sources

The point is not the exact tools. The point is the discipline: each stage exists because a naive approach
failed in a specific way.

---

## Result: Answers Become Debbuggable

Once the query layer returns sources with the answer, two things change:

- disagreements become diagnosable (“the evidence was missing / wrong / out of date”)
- improvements become measurable (you can attribute changes to retrieval stages, not vibes)

Proof points to capture before publishing:
- [NEEDS_DATA: one before/after example where higher top_k made output worse]
- [NEEDS_DATA: one concrete retrieval stage change and measurable impact, with run conditions]

## Limits / when not to use

- Evidence shaping improves consistency, but it cannot recover missing or stale source material.
- Larger `top_k` can still be useful for exploration; the point is to shape and bound, not to forbid.
- Rerankers add latency; they pay for themselves only when they reduce review or retry cost.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; `query.use_hybrid_search=true`; evidence shaping via `query.reranker_retrieve_k`, `query.reranker_top_k`, `query.use_mmr`, `query.use_lost_in_middle`; token budgets `ollama.context_window=32k`, `ollama.num_predict` capped
- Dataset class: internal governance handbook (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## The Lesson

If your RAG system feels unreliable, don’t start by “prompting harder”.

Start by treating retrieval as a pipeline:

- complementary retrievers
- explicit fusion
- deliberate context shaping
- sources returned as a first-class output

That’s the path from “prototype answers” to “systems people can trust”.
