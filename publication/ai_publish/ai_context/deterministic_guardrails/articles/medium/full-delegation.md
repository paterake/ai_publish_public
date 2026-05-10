# You Don't Have to Understand Every Line the Agent Wrote

The AI authored the PRDs. It sequenced the publication workflow. It drafted this article.

I defined the direction. The harness did the rest.

If you've spent time in discussions about agentic coding, that setup probably triggers a concern: what happens to your understanding when you delegate execution entirely? If you didn't write it, how do you know it's correct? What happens when something breaks and you have to debug output you never touched?

These are good questions. The answer isn't to delegate less. The answer is a different question: what governance system makes full delegation safe?

---

## The Failure Mode Is Real

When agentic coding goes wrong, it goes wrong quietly. The agent touches dozens of files. The engineer skims the diff. Understanding leaks away — not in one bad commit, but across a hundred reasonable ones. The internal model of the system diverges from the repo. Six months later, something breaks in production and nobody knows where to start.

This is the failure mode engineers are right to worry about. And it happens precisely when execution delegation happens without a governance layer.

The failure isn't delegation. The failure is delegation without contracts.

One concrete example from this workflow: early on, an agent created a new publication module without registering it in the tracking system. Nothing broke loudly. A coverage check ran three sessions later, assumed everything was registered, and silently skipped the invisible module. The gap accumulated for two sessions before a contract check caught it.

That's the silent failure mode. The fix isn't to watch more carefully. The fix is to make that failure structurally impossible.

---

## What Governance-First Delegation Looks Like

In this publishing workflow, the AI assistant is an execution engine. I define creative intent — what to publish, what the story is, what each module is for. Everything else — PRDs, article drafts, publication sequencing, gap analysis, content review — is handled by the harness.

That is full execution delegation. Not "let AI write the first draft." Full delegation.

It is safe because of four engineering decisions made before any delegation happened.

---

## Single Sources of Truth — With Generated Projections

The dangerous case with agentic workflows isn't one bad decision. It's two sessions making locally reasonable decisions from different slices of state — and nobody noticing the divergence until it compounds.

The fix: every type of state has one canonical home. Generated outputs (publication ordering, coverage tables) are produced from those sources by deterministic scripts. The agent doesn't infer sequencing from prose. It reads a projection.

This resolves one specific failure: "I don't know what the system currently believes."

---

## Executable Contracts

AI output has uneven quality in ways that are hard to predict. An agent can produce a well-structured PRD and quietly miss a required field. It can draft an article that passes a skim review but contains an internal path that makes it unpublishable. It can create a new module without registering it in tracking — invisible to every downstream workflow.

These are governance failures, not capability failures. And they're silent unless you make them loud.

A repo contract checker runs as a hard gate: required structures must exist, invariants must hold, tracking must be current. The agent's output hits a deterministic net. Only correct output passes.

This resolves: "How do I know the output is actually right, not just plausible?"

---

## Bounded Context as a Structural Constraint

Large contexts are risky. When an agent loads the entire repo to do a local task, it starts making decisions from irrelevant state — and starts "helpfully" fixing things that weren't in scope.

The workflow has an explicit session model. Each stage loads only what it needs. Source sync is human-triggered. Gap analysis reads pre-computed outputs, not the whole repo. Drafting reads one PRD and the listed sources.

The guardrail is architectural: the workflow is designed so the agent never needs global recall to do local work.

This resolves: "What stops the agent from touching things it shouldn't?"

---

## Procedures Encoded as Skills

If you re-derive "how we do things" every session, you're asking a non-deterministic component to reliably reproduce a deterministic procedure. That's where consistency breaks down across accumulated sessions.

Repeatable operations — onboarding a new module, drafting a publication pair, reviewing a draft against source material — are encoded as skills: explicit step sequences that reference canonical standards rather than restating them. Any session, any assistant, same procedure.

The encoded skill is the institutional memory. Not the session. Not the assistant's recall.

This resolves: "What happens to consistency across dozens of sessions with different assistants?"

---

## The Distinction That Actually Matters

There is a widely-repeated principle in agentic coding: never delegate work you couldn't do yourself. The concern behind it is real. But as a rule, it caps your leverage at what one human can execute.

The distinction that scales is different: you don't need to understand every line the agent wrote. You need to understand the system the agent operates within.

Those are not the same thing. One requires you to follow every implementation detail. The other requires you to design good constraints — and trust them.

You can't hand-author every PRD in a publishing workflow spanning multiple modules and dozens of articles. But you can design a harness that produces correct PRDs reliably. The understanding lives in the governance system, not in your head.

---

## What You Retain

When you build this way, what stays with you is:

- **Creative intent** — the direction, the story, the judgement about what matters
- **System design** — the governance model, the contracts, the session model
- **Evaluation capacity** — the ability to recognise when something is wrong, even if you didn't produce it

What you hand off is execution. The harness holds the implementation. The governance system holds the constraints. Your job is to design both — and then direct.

That is leverage. Not dependency.

---

**Related:** [Designing Guardrails for AI Coding Assistants](deterministic-guardrails.md) covers the mechanism behind this system — why each guardrail is designed the way it is.
