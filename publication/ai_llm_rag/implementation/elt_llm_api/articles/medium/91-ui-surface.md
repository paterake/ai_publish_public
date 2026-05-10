# A RAG UI That Preserves Evidence

A chat box without sources is a demo. If the UI hides evidence, trust collapses on the first challenge. This explains why citations must be a first-class UI element in any enterprise RAG system.

The only usable interface is a command line.

The second-most common failure is worse:

The UI exists, but it hides evidence. Users see an answer, not the sources. Trust collapses the first time the
answer is challenged.

This article explains the design principle behind a simple RAG UI: the interface must preserve auditability,
not paper over it.

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

## Why Adoption Requires a Surface fails at scale

A strong query engine is necessary. It is not sufficient.

Non-technical users won’t run a Python CLI. They won’t manage profiles. They won’t inspect JSON outputs.

If the system is meant to be used broadly, it needs a surface that is:

- discoverable
- easy to operate
- evidence-first

---

## What most teams try first: “Just Add a Chat Box”

Adding a chat box is easy.

Preserving trust is not.

The naive UI shows:

- a question
- a fluent answer

It does not show:

- which knowledge base was queried
- which sources were used
- what the system did when evidence was missing

That’s a demo interface. It’s not an adoption interface.

---

## How the solution works: Make Sources a First‑Class UI Element

The interface design is shaped by one rule:

If the answer cannot be justified, it is not usable.

So the UI must surface:

- the active profile (what data is being searched)
- the answer
- the citations/sources

This also makes the system teachable: users learn what kinds of evidence exist and how queries map to sources.

---

## Result: A UI That Enables Trust

With evidence visible, two things happen:

- users can validate answers quickly
- disagreements become actionable (“this source is wrong/out of date”) instead of subjective

Proof points to capture before publishing:
- [NEEDS_DATA: publish-safe screenshot set from a public dataset run]

## Limits / when not to use

- A UI cannot compensate for a weak retrieval pipeline; evidence-first interfaces still need good evidence.
- Screenshots and demos must be built on public-safe datasets; do not publish internal runs.
- Adoption work includes onboarding and trust-building; a UI is necessary, not sufficient.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; UI loads a RAG configuration profile (vector store persist dir, inference settings, query knobs) and a query profile (collection prefixes + optional overrides)
- Dataset class: internal governance handbook (anonymised) for local usage; publish-safe UI screenshots should use public dataset equivalents
- Non-reproducible from this article: exact prompts, proprietary taxonomies, and internal contracts

---

## The Lesson

If you want adoption, build a surface.

If you want trust, make evidence visible in that surface.
