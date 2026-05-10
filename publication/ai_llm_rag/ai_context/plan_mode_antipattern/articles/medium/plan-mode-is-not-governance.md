# Plan Mode Isn’t Governance (And Governance Is What Scales)

“Plan first” is good advice.

Treating a session permission mode as your primary safety mechanism is not.

If you work with AI coding assistants on anything larger than a one-off script, your dominant risks aren’t “the assistant started coding too early”. They’re cross-session drift, cross-assistant inconsistency, and the slow dilution of constraints as chats get long and summarised.

Plan Mode helps inside a session. Governance is what survives across sessions.

This article explains the difference, why it matters, and what to build instead.

## What Plan Mode Actually Solves

In the most common workflow, planning lives inside chat:

1. Ask for a change
2. The assistant proposes an approach
3. You confirm or correct it
4. The assistant implements

Plan Mode is a useful optimisation here because it reduces wasted write/undo cycles. It forces the assistant to stop and think before it edits files.

If your only control is “human reviews the diff”, a read-only planning phase is sensible.

## The Failure Modes Plan Mode Doesn’t Touch

Plan Mode is fundamentally session-scoped. It does not solve:

- **Cross-session drift**: a later session reintroduces an old pattern because the assistant “forgot” the constraint
- **Cross-assistant inconsistency**: different tools produce different architectural styles because their “memory” isn’t shared
- **Compaction dilution**: the longer the chat, the more subtle constraints are summarised away
- **Token burn by context bloat**: loading “everything” lowers precision and increases cost

These are the problems that break long-lived AI-assisted work.

If you rely on a single chat as the system of record, you will keep re-planning the same work, re-discovering the same constraints, and re-fixing the same regressions.

## Planning Is a Principle; Where It Lives Is a Design Choice

This is the distinction most “plan first” guidance does not make:

- Planning is a principle: understand before changing
- Plan Mode is one implementation surface: chat-state gating

If you want planning to scale, you need a durable surface that outlives the session.

That surface is not a mode. It’s the repository.

## What Replaces “Plan Mode as Default”

Governance-first AI development treats planning as a system property enforced by four things:

### 1) A durable contract layer

You need a small set of documents that define how work must be done:

- the non-negotiable pillars and boundaries
- the workflow triggers (what must be updated when)
- the conventions that prevent architectural drift

This is not “documentation for humans”. It is the durable steering mechanism that keeps any assistant convergent across time.

### 2) PRDs that capture intent, not approach

PRDs rot when they describe implementation strategy. A future assistant reads the stale “how”, reintroduces it, and you get inconsistency across modules.

PRDs stay durable when they capture:

- why the module exists
- what constraints must hold
- what outputs/contracts define “done”
- what evidence makes claims publishable

The approach belongs in the shared governance layer, maintained once, not duplicated per module.

### 3) Executable gates

A rule without a failure signal is a preference.

If you care about preventing drift, encode it as something that fails fast:

- a check that prevents accidental broad rewrites
- a gate that enforces doc alignment
- a drift test that stops “summary truth” diverging from reality

This is what makes governance automatic. It does not rely on an assistant remembering what you meant last week.

### 4) A session model designed for long backlogs

Long-running work is not best done in one never-ending chat.

A better operating model is:

- new session per bounded backlog item
- durable checkpoint notes written into the backlog document
- accumulated constraints explicitly forwarded

Planning still happens. It’s just captured in a form that survives.

## The Practical Test

If you can close every chat session, start a new one with a different assistant, and still get consistent work by pointing at the repo contracts, you have governance.

If you cannot, you are relying on session memory and implicit planning.

Plan Mode can still be useful as a technique in specific moments:

- when you are introducing a new invariant that doesn’t yet have a gate
- when you are genuinely unfamiliar with an area and need a map before choosing a design

But as a default posture, a session permission mode is a weak place to store safety.

If you want autonomy, you want a system where the plan is already in the files.

## The Point

Plan first is correct.

Planning trapped inside a session is not durable.

If you want AI assistants to work safely across weeks, across tools, and across machines, build governance that survives the session.
