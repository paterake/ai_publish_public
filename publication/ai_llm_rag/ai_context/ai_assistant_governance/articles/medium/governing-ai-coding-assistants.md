# Governing AI Coding Assistants: How to Make Delegation Repeatable

Most teams try to govern AI coding assistants by writing better prompts.

That works until the work spans multiple sessions, multiple assistants, or multiple people. Then the dominant failure mode isn’t “the assistant didn’t understand”. It’s that the system has no durable control surface.

This article is about governing AI-assisted development as an engineering system: contracts, constitution, procedures, and gates.

The goal is not to reduce what the assistant can do. The goal is to make autonomy safe, repeatable, and portable across tools and sessions.

## The Operating Contract: Human ↔ Assistant

If you want to delegate more than toy tasks, the roles must be explicit:

- **Human role**: define the problem, provide domain context and data, run commands when needed, and make acceptance decisions
- **Assistant role**: design and implement changes that satisfy the contracts and repo standards

The trick is the enforcement mechanism:

> The assistant does not “stay correct” by remembering. It stays correct by passing gates.

This is what makes the model portable. If you switch assistants, you are not switching the rules. You are switching the executor.

## The Four-Part Model: Contracts + Constitution + Procedures + Gates

Think of governance as four layers, each with a different responsibility.

### 1) Contracts (module-level)

Each module needs a clear entry point contract.

The contract should be sufficient for an assistant to implement or modify the module without relying on long chat history. It defines intent, scope, boundaries, and what “done” looks like.

The contract should not be a copy of the implementation. If the contract becomes “whatever the code currently does”, you’ve lost governance.

### 2) Constitution (repo-level)

Cross-cutting rules must live in one place.

If you scatter core rules across PRDs, you guarantee drift: different modules will encode different eras of the same principle.

The constitution is where you keep:

- the non-negotiable pillars
- boundary rules (what belongs where)
- workflow triggers (what must be updated when)
- quality and publication standards

### 3) Procedures (repeatable)

Procedures exist to avoid “go and remember how we do this”.

If a check must happen every time, encode it as a repeatable workflow: what to read, what to update, what order to do it in.

Procedures reduce the cognitive load on both the human and the assistant. The goal is consistent execution.

### 4) Gates (automated)

This is what makes the system self-correcting.

A governance rule without a failure signal is advisory. Advisory rules fail under task pressure.

Gates turn your governance into constraints:

- the build fails when drift appears
- the assistant has an unambiguous signal that “done” is not true yet
- the human doesn’t need to police every rule manually

## Why This Works With Short Sessions and Different Assistants

Most advice assumes one assistant and one long conversation.

That is a fragile architecture. It depends on conversational continuity and tool-specific memory.

A governance-first setup is designed to be restartable:

- durable state lives in git-backed contracts and documents
- procedural state lives in the workflow
- correctness is proven by gates

This enables an intentional operating model:

- end sessions aggressively
- start fresh sessions for new tasks
- reload the authoritative contracts at the start

When “memory” is externalised into the repo, you stop relying on any one assistant’s internal state.

## A Practical Boundary Rule: Don’t Duplicate Shared Operational Contracts

One of the easiest ways to create drift is to duplicate shared rules inside module specs.

If a shared capability exists (observability conventions, run artefacts, error envelopes), modules should inherit it by reference and only add what is genuinely module-specific.

This keeps:

- the shared contract maintainable (one place to update)
- module PRDs lean (intent-led)
- the assistant’s context high-signal (less repetition, fewer contradictions)

## The Outcome: Autonomy You Can Trust

When governance is treated as a system, not as prompting technique, you get:

- consistent behaviour across sessions
- consistent behaviour across assistants
- recoverability after a machine change
- faster execution because the assistant isn’t re-deriving rules every time

Most importantly, you can delegate without gambling.

The human’s job shifts from “supervise every output” to “design the system that makes outputs verifiable”.

That is what it means to govern AI coding assistants.
