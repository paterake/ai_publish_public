# The Backlog Continuity Contract: How to Make AI Work Resumable Across Sessions

The biggest lie in AI-assisted development is that chat history is a system of record.

It isn’t.

If a piece of work spans multiple sessions, the most common failure modes aren’t about capability. They’re about continuity:

- constraints decided earlier are forgotten later
- “done vs todo” state is unclear, so work is redone
- long chats accumulate noise and subtle rules get diluted
- a different assistant can’t pick up the thread without rereading everything

This article describes a simple pattern that fixes that: a backlog continuity contract.

It’s a way to write backlog documents so any assistant can resume from a cold start in one line, without relying on previous conversations.

The idea is simple: treat the assistant as stateless by default, and make the system stateful through durable artefacts.

One thing that’s easy to miss: in a harnessed workflow, this is not a discipline a human “keeps up”.
The harness owns the continuity loop. Updating the anchor document is part of the assistant’s completion step for each item.
No human clerk. No copy/paste summaries between sessions. If the anchor doc isn’t updated, the work isn’t done.

## The Cynic’s Objection (And Why It Misses the Point)

You can hear the pushback coming:

- “Just get a bigger context window.”
- “Just buy a bigger machine.”
- “Just work in smaller pieces.”

All three miss the actual failure mode.

This pattern is not a workaround for limited compute. It is a governance solution for a non-deterministic collaborator.

Bigger context does not give you durable state. It gives you more transient text to interpret. The hard problems that break multi-session work are not about whether the assistant can physically read more. They’re about whether the system can reliably preserve:

- what is done vs still in progress
- which constraints remain active
- how to verify correctness

“Work in smaller pieces” is directionally right — and it is exactly what this contract assumes. The difference is that without a durable hand-off, “piecemeal” work still leaks constraints and forces re-triage in the next session. Stable anchors plus minimal hand-offs make small-slice work resumable without relying on memory.

## This Is a Collaboration System, Not a Prompt

Most advice about AI coding assistants treats the unit of design as a prompt: write better instructions and you get better outcomes.

The unit of design here is the collaboration.

Once work spans multiple sessions, you are building a system with:

- **state** (done vs todo, decisions already made)
- **constraints** (what must not change)
- **verification** (how you know it’s correct)
- **handoffs** (how the next session resumes without re-triage)

If those things live only in chat history, you don’t have a system of record. You have an improv transcript.

Anchors and hand-offs turn collaboration state into durable artefacts — and that is why the pattern works.

## The Operating Model: New Session Per Backlog Item

The counterintuitive part of this pattern is that it assumes short sessions on purpose.

For long-running backlogs, the intended workflow is:

1. Use a new session for each bounded backlog item
2. Update the backlog document at the end of the item
3. Let the next session resume from the updated document

This is not a fallback for when a chat gets too long. It is the primary operating model.

Why it works:

- a fresh session attends to the loaded context at full precision
- item-specific constraints don’t leak into the next item as residual context
- you stop treating “remembering what we did” as the assistant’s job

The document becomes the memory surface.

In practice, the “end of session” protocol is automated:
the outgoing assistant writes the hand-off back into the anchor doc as structured state (done / constraints / next / verification), so the next session can start from the document alone.
You can review the diff if you want to, but continuity does not depend on you doing anything manually.

## The Key Insight: Chat History Is Not Durable State

Chat transcripts don’t scale for either humans or assistants:

- humans can’t efficiently reconstruct state from many hours of conversation
- assistants can’t reliably infer which old statements are still true
- critical constraints disappear simply because they aren’t repeated

If correctness depends on constraints across time, those constraints must be written somewhere durable.

## The Pattern: One Anchor Document Per Thread

Choose exactly one anchor document per backlog thread.

Examples:

- cross-module refactor: a single root tracker
- module enhancement backlog: a module TODO
- contract gaps: the module PRD

Do not maintain the same plan in multiple places. Link instead.

The anchor document is the durable thread state.

## Tracking Crumbs: The Minimal Hand-Off That Makes Resumption Work

At the end of each backlog item, the outgoing assistant writes “tracking crumbs” for the incoming assistant.

This is not a narrative summary and it is not “notes for the human”.
It is a structured hand-off written specifically so the next assistant can resume from a cold start, and the harness can keep the thread coherent without a person reconstructing context from chat.

The crumbs answer four questions:

1. **Where did we get to?** (what changed, at file level)
2. **What must not change?** (constraints that remain active)
3. **What comes next?** (the next ordered action)
4. **How do we know it’s correct?** (the exact verification step)

Write crumbs as if briefing a capable colleague who has never seen the prior chat.

Precise. Factual. Minimal.

## Constraint Inheritance: The Part Most Backlogs Miss

Continuity is not only “what’s next”.

Every completed item establishes constraints that future items must honour. Those constraints accumulate.

If you do not explicitly forward them, a future assistant has no reason to apply them.

This is where drift appears: not because the assistant is careless, but because the constraint was never stated in the next item.

The rule is simple:

> If a constraint remains active after an item completes, it must be written into the anchor document as an inherited constraint.

## The One-Line Resume Prompt

When the anchor document is maintained, resuming becomes a single line:

> From `<anchor_doc>`: continue `<item anchor>`.

You are no longer asking the assistant to remember. You are asking it to read.

## Why This Makes Each Session More Productive

This pattern does three things simultaneously:

- reduces token burn (only the active slice needs to be loaded)
- increases correctness (constraints are explicit, not inferred)
- makes multi-assistant work coherent (any tool can read the same anchor state)

It also changes the human experience.

Instead of “keep a long chat alive so we don’t lose context”, the operator experience becomes:

- pick the next item
- start a fresh session
- (optionally) review the diff

The system stays coherent because the backlog document is the continuity mechanism.

## If You’re Starting From Scratch

If you want to retrofit this to an existing tracker, start with:

1. Add a short “Next up” list that points to stable anchors
2. Ensure each backlog item has a stable anchor ID
3. Add a hand-off template under each item (done, constraints, next, verification)

After that, the pattern becomes self-reinforcing: each completed item writes the scaffolding that makes the next item easy.

The result is a workflow that scales across sessions, across assistants, and across time.

## Limits / When Not to Use

- If the work is a single, one-hour task, the overhead is not worth it.
- If you refuse to update the anchor document, the system has no durable memory surface to resume from.
- If you maintain multiple competing trackers, the pattern collapses into drift again.
