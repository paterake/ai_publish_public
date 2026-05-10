# Designing Guardrails for AI Coding Assistants

I wasn’t worried about one catastrophic AI mistake.

I was worried about slow dilution.

A strong system, gradually regressing as it passes through dozens of sessions and multiple assistants — each working from a different slice of context, each making “reasonable” edits that are locally correct but globally erosive.

That failure mode is harder to detect than a single bad change. A large, obviously wrong commit can be reverted. A slow steady loss of constraints across a hundred commits is harder to recover from, because you won’t have a clean memory of which change introduced the drift — and the assistant won’t either.

The fix wasn’t better prompting. The fix was treating AI behaviour governance as a software engineering problem: deterministic boundaries, executable contracts, and a control plane that survives any single session.

This is the pattern.

---

## Key takeaways

- Non-deterministic assistants need deterministic boundaries, not longer prompts.
- Make drift loud: encode invariants as executable checks, not “best effort”.
- Keep state small and explicit: one source of truth, plus generated projections.

## Who this is for

This is for engineers using AI coding assistants on real repos and workflows, where “review everything” stops scaling once an assistant can touch tens or hundreds of files per session.

## The Real Failure Mode Isn’t Bad Code

Most discussions about AI coding assistants focus on correctness: does the code compile? Does it pass tests? Did it misunderstand the requirement?

Those matter, but they miss the failure mode that makes people uneasy delegating real work over time:

- the assistant edits more than you asked
- it updates code but not the docs that make the code usable
- it “simplifies” by deleting details that were load-bearing
- it assumes a file is derived and safe to regenerate, when it is actually the only copy

These are governance failures, not capability failures.

The risk compounds because the pace of change accelerates. Once an assistant can touch tens or hundreds of files in a session, “review everything” stops being a realistic safety strategy. The only sustainable posture is to enforce the invariants that must never drift — and make failures loud.

A non-deterministic agent will occasionally take a route you wouldn’t. That’s not a bug in the model. It’s a property of the system. The question is whether your workflow makes those routes harmless, detectable, or impossible.

---

## Why Better Prompting Doesn’t Solve It

You can write excellent instructions. You can tighten your review loop. You can add more rules.

And it will still regress, because:

1. The assistant’s working context is limited and session-scoped.
2. Tool memory (if it exists) is vendor-controlled and not portable across assistants.
3. The behaviour you want is a system property, not a sentence-level property.

If you want predictable outcomes from non-deterministic components, you don’t write longer instructions. You design control surfaces and invariants — so the system remains stable even when the assistant is working from partial context.

---

## The Shift: Treat the Harness Like a System

In this publishing workflow, I treat the AI assistant as an execution engine that operates inside a deterministic framework.

The framework has four ideas:

1. **Deterministic control plane:** there is a single source of truth for each type of state.
2. **Executable contracts:** invariants are checked by scripts, not enforced by memory.
3. **Bounded scope:** each task loads only the context it needs, and nothing else.
4. **Procedures over prose:** repeatable operations are encoded as skills and scripts, not rediscovered each session.

When these are in place, the assistant can still be non-deterministic in its writing style or implementation choices, but it cannot silently violate the collaboration contract without being caught.

---

## Guardrail 1: Single Sources of Truth + Generated Projections

A common way to “track” a process is a big document that contains everything: status, ordering, metadata, decisions, logs, and drafts.

That works for humans. It is expensive and fragile for AI.

Instead, this workflow uses a simple pattern:

- store durable state in small, structured pack PRDs
- generate a compact projection for the one question you ask repeatedly: “what’s next?”

Concretely:

- each publication pack has a PRD that owns its status and publishing order
- a deterministic script reads every PRD and generates a short publication path table

The assistant doesn’t infer sequencing by reading prose. It reads the projection.

This is a safety control (you avoid divergent copies) and a token control (you avoid rereading a monolith to answer a cheap question).

---

## Guardrail 2: Executable Contracts (Not Vibes)

The most important guardrail in the repo is not a file of advice. It is a contract checker.

A repo contract checker enforces invariants such as:

- the publication workspace must be registered and have entrypoint files
- packs must be registered in the project’s publisher path so they are not “invisible”
- assistant-facing skills must be orchestration-only (they reference canonical standards instead of duplicating them)
- repo docs must not leak local absolute paths
- publishable drafts must not include internal repo paths that assume the reader has the repository open
- required workflow and governance files must exist

This matters because AI-assisted work fails in quiet ways. The dangerous case is not a loud error. It’s a subtle drift: duplicated rules in two places, a missing pack registration, a new project directory created without the required tracking files, a path leak that makes a draft unpublishable.

Executable contracts convert those into hard failures you can’t ignore.

---

## Guardrail 3: Bounded Context as a First-Class Design Constraint

Most assistants behave as if “load everything” is safe.

It isn’t.

Large contexts are not just expensive. They are risky: irrelevant state crowds out relevant state, and the assistant starts making decisions from the wrong slice of reality.

So this workflow has an explicit session model that constrains what is loaded when:

1. source sync (human-triggered when needed)
2. gap analysis (reads pre-computed outputs, not the whole repo)
3. sequencing (edit PRDs, regenerate the projection)
4. drafting (read the projection, one PRD, and the listed sources)

The guardrail here is architectural: you design the workflow so the assistant never needs to load global state to do local work.

Token efficiency is not the primary goal. It is the symptom of good boundaries.

---

## Guardrail 4: Procedures Encoded as Skills

If the assistant has to re-derive “how we do things” every session, you are depending on a non-deterministic component to reproduce a deterministic procedure.

That’s backwards.

In this workflow, repeatable operations are encoded as skills and scripts:

- onboarding a new module into the publication workspace (one folder per project/module)
- drafting a Medium/LinkedIn pair from a PRD
- reviewing drafts against source material
- syncing publication packs after source changes

Skills are not a second copy of standards. They are orchestration: “read the canonical rule here, then do the steps”.

That division is a governance control (one canonical home for standards) and a scaling control (any assistant can execute the same procedure).

---

## What You Get When You Build This Way

You don’t get perfect behaviour. You get something better:

- the assistant is allowed to be non-deterministic inside a safe sandbox
- destructive or drifting behaviour is prevented by structure, or caught by contracts
- state lives in places that are version-controlled and portable across assistants
- each session loads less context because the workflow doesn’t demand global recall

That last point is the surprising one. The cheapest workflow is often the safest one, because both require the same discipline: single sources of truth, explicit boundaries, and deterministic checks.

---

## The Lesson

If you want to delegate to AI coding assistants, treat governance as engineering.

Don’t fight non-determinism with better instructions.
Contain it with deterministic guardrails at the boundary.

Question: what is the one failure mode that would make you stop delegating to an AI assistant — and what guardrail would make it harmless?
