# Ingestion Is Where RAG Quality Is Won

Most RAG quality problems start at ingestion. If you destroy structure upstream, you can’t prompt it back later. This is why ingestion must be treated as a substrate decision, not just plumbing.

When an answer is wrong, teams reach for prompts, rerankers, and bigger models. Those help, but only after the
retrieval substrate is correct. If ingestion turns structured material into fragmented prose, no amount of
synthesis will reconstruct what you destroyed upstream.

This article is the story of building an ingestion layer that treats different source types differently and
produces the right substrate for each.

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

## Why Heterogeneous Sources Don’t Behave Like Text fails at scale

Enterprise knowledge is not one format. It is:

- layout-heavy documents
- semi-structured tables
- structured models and inventories

If you treat everything as “chunks to embed”, you end up with:

- broken semantic units (tables split mid-row)
- loss of hierarchy (headings and sections flattened)
- higher downstream cost (more chunks, larger prompts)
- lower trust (answers feel random because evidence is fragmented)

---

## What most teams try first: Put Everything Into RAG

The naive design is simple:

1. convert every source to text
2. chunk it
3. embed it
4. retrieve and synthesise

It works until you hit structured sources.

For many inputs, retrieval is not the right primitive. If you already have a schema (or can infer one safely),
the best representation is deterministic JSON, not probabilistic search over prose.

---

## How the solution works: A Dual Store + Deterministic Sidecars

The ingestion design is a substrate decision:

- **Unstructured sources** → chunks + embeddings + dual retrieval substrate (dense vectors + sparse docstores)
- **Structured sources** → deterministic JSON sidecars (and optional retrieval text when it is genuinely useful)

This gives downstream modules two options:

- retrieve evidence when the answer lives in text
- look up a record deterministically when the answer is a field

That boundary is the anti-hallucination posture: use deterministic artefacts when possible.

---

## Result: Downstream Modules Get Better Inputs

Once ingestion produces coherent chunks and deterministic models, downstream behaviour improves:

- fewer “lost in the middle” failures because context is shaped from coherent units
- fewer hallucinations because structured facts are retrieved deterministically
- lower runtime cost because the retrieval substrate is smaller and more relevant

Proof points to capture before publishing:
- [NEEDS_DATA: one publish-safe example showing table-aware chunking vs naive chunking]
- [NEEDS_DATA: cost/throughput comparison before/after ingestion changes]

## Limits / when not to use

- Layout-heavy PDFs remain hard; even strong parsers can lose tables and hierarchy.
- Chunking is a trade-off (coherence vs recall vs cost); defaults must be justified by real runs.
- Deterministic sidecars require a stable schema; if the schema changes weekly, you will churn.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; Docling PDF preprocessor (`table_format=markdown`, `split_by_sections=true`, tuned `section_splitting.*`); chunking `strategy=table_aware` with bounded `chunk_size`/`chunk_overlap`
- Dataset class: internal governance handbook PDF + enterprise architecture model exports (anonymised; no client identifiers); public PDF equivalents for publish-safe examples
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## The Lesson

Ingestion is not plumbing. It is the foundation of retrieval quality.

If you want reliable RAG, do not start at the model. Start at the substrate:

- what is this data type?
- what is the right representation?
- what is deterministic, and what must be probabilistic?

That is how you build a system that stays credible as it scales.
