# The PRD as an AI-Assistant Contract

*A PRD written as “documentation about the code” cannot govern an AI coding assistant.*

Most PRDs are written for a human reader.

They assume the reader will:

- infer intent from context
- fill in gaps with experience
- open the code when something is ambiguous

That assumption breaks the moment an AI coding assistant becomes part of the delivery team.

An assistant doesn’t “read between the lines”. It executes what you described. If your PRD is a loose brief, you get a loose implementation. If your PRD is a contract, you get something you can govern.

This article is about the structural change:

**A good AI-era PRD is not documentation about the current code. It’s a contract for what must be true — independent of how the code achieves it.**

---

## Key takeaways

- A PRD that lists files and classes is not a spec. It’s a memory aid for humans.
- AI-ready PRDs are contract-first, code-blind, and boundary-explicit.
- The most important line in a PRD is often what it refuses to own.

---

## The failure mode: “documentation about code”

If you have ever written a PRD that says something like:

- “add a new class called X”
- “update file Y”
- “add tests under Z”

you wrote something that can work for a human team, because humans understand what those instructions are *really trying to achieve*.

For an AI assistant, those instructions are a trap.

They anchor the assistant on artefacts (files, classes) instead of outcomes (contracts, invariants). The assistant optimises for producing the named artefacts, and the system drifts away from the real requirement.

The deeper problem is that file-centric PRDs silently couple the specification to the current implementation. That coupling is exactly what makes refactors dangerous: the spec changes when the code changes.

Contracts shouldn’t do that.

---

## What an AI-ready PRD must enable

Here is the standard I hold PRDs to when an assistant is involved:

The PRD must describe the module intent, scope, contracts, and boundaries well enough that an assistant could rebuild the module’s functionality without opening implementation files.

That sounds extreme until you notice what it forces:

- You must be explicit about inputs and outputs.
- You must state what “correct” means.
- You must name the failure envelope.

In other words: you must write the things that matter even if the code is rewritten.

---

## Five non-negotiables

### 1) Contract-first

Describe what the module guarantees:

- inputs (schemas, required fields, validation)
- outputs (artefacts, schemas, stability promises)
- invariants (what must always be true)

Avoid narrating the implementation.

When an assistant changes the implementation, the contract remains stable. That stability is the governance surface.

### 2) Code-blind

Do not reference implementation file paths, function names, or unit test files.

Those are artefacts. The code is one possible realisation.

If a PRD can only be understood by opening the repo, it is not governing the repo. It is describing it.

### 3) Boundary-explicit

State what the module owns vs what it delegates.

This is where most PRDs fail, because boundaries are uncomfortable. They force trade-offs. They force you to say “no”.

But boundaries are where systems stay coherent.

If you don’t state boundaries, you get duplication: every module implements its own “helpful” version of the same primitive. That looks productive in the short term. It becomes drift in the long term.

### 4) Determinism where promised

If you claim deterministic outputs, state the conditions.

“Deterministic” is not a vibe. It is a contract:

- same inputs + same config ⇒ same outputs
- if the module calls an LLM, name what makes it deterministic (or explicitly state where it is probabilistic)

Without conditions, “deterministic” becomes marketing language. That destroys trust.

### 5) Trust posture

State caps and safety boundaries:

- what is rejected vs degraded vs retried
- what constitutes “insufficient evidence”
- what happens when a dependency is unavailable

For AI-assisted systems, trust posture is not optional. It is the difference between a tool and a liability.

---

## The boundary rule most teams miss

One rule prevents a huge amount of drift:

**Non-core modules must not re-define shared platform plumbing.**

If you have a shared platform layer — observability primitives, run context, manifests, evaluation hooks — the contract for those lives once. Every other module inherits it.

PRDs that duplicate shared plumbing create two sources of truth. Two sources of truth is drift with a delay.

The PRD should name inheritance in one sentence and move on.

---

## A minimal PRD shape that works

If you want a practical template, here is the minimal shape that consistently governs assistants:

1. **Problem**: what pain exists and why it is hard
2. **Solution**: one paragraph, high level
3. **Scope and boundaries**:
   - owns
   - delegates
   - out of scope
4. **Contracts**:
   - inputs
   - outputs
   - error envelope
5. **Requirements**:
   - functional requirements (SHALL statements)
   - non-functional requirements (caps, determinism conditions, safety)
6. **Acceptance criteria**: verifiable checkboxes tied to the contracts
7. **Known limitations**: honest, explicit, bounded

This looks more formal than many PRDs. In practice it’s less work than “describe the implementation”, because it forces you to write only the information that survives change.

---

## Close

An AI coding assistant doesn’t make requirements less important.

It makes weak requirements visible.

If your PRD is a brief, the assistant will treat it as a brief. If your PRD is a contract, the assistant has something stable to build against, refactor against, and be held accountable to.

There is a deeper reason why this matters now more than before. In traditional development, code was the primary artifact — the PRD described what the code should do, but the code was what shipped. In AI-assisted development, this inverts: code is generated on demand from direction. The PRD is the primary artifact. It is the thing that cannot be regenerated without human effort — the encoded intent, the stated constraint, the invariant that must hold regardless of how the implementation is rewritten. Code is the by-product.

Write the PRD accordingly: as the thing that outlasts any implementation, not as a description of the one you currently have.
