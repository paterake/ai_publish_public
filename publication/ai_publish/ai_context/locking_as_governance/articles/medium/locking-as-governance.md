# Lock the Draft: Governance for Non‑Deterministic Editing

The easiest way to ruin a great technical draft is to let an assistant “improve the writing”.

Not because the assistant is malicious.

Because prose has load‑bearing details, and assistants are very good at deleting them while believing they are making the piece clearer.

## Key takeaways

- Any tuned AI output is an asset: treat it as something you protect, not something you regenerate.
- “Better wording” can silently delete constraints, proof points, and limits — the credibility layer.
- Lock tuned outputs and require surgical edits, or you will re-author by accident.

## Who this is for

This is for anyone producing AI-assisted outputs where the hard part is not generating text, but preserving the specific details that make the output worth trusting. Publishing is the worked example here, but the same failure mode applies to any tuned AI output: governed modules, calibrated configurations, finalized reports.

## The failure mode: polished, but weaker

Most people worry about AI assistants doing something obviously wrong: fabricating a metric, shipping a broken change, leaking a private identifier.

There is another failure mode that is harder to spot, because it looks like success:

- the writing becomes smoother
- the structure becomes more “blog-like”
- the awkward detail gets rephrased away
- the caveat disappears

The piece reads better.

It also says less.

In any AI-assisted output, the most precise statements are often the most fragile: the sentence that states the constraint, the one that names the failure mode, the one that admits a limit. Those lines are easy to remove because they feel like friction.

That friction is the credibility layer.

## Why assistants cause this (even when they are “helpful”)

Assistants optimise for local coherence.

They smooth repetition. They tighten paragraphs. They unify voice. They remove what looks redundant.

But the details you need to keep are often not redundant. They are redundant‑looking because they are safeguards:

- You repeat the constraint because it changes the reader’s interpretation of the result.
- You repeat the proof point because it separates “I built something” from “this worked under conditions”.
- You repeat the limit because readers over‑generalise.

A rewrite that removes those details can still look reasonable line by line. The damage is global: the article becomes easier to skim and easier to dismiss.

## The decision: treat tuned outputs like locked artefacts

If you want a workflow that stays reliable across sessions and across different assistants, you need a state transition for any AI-assisted output:

- Draft (allowed to change a lot)
- Tuned (high quality; pacing and framing are deliberate)
- Locked (changes must be surgical)

Locking is not a promise that “this will never change”.

It is a promise that changes will not silently become re-authoring.

## What “locked” means in practice

When a draft is locked:

- you do not regenerate it from scratch
- you do not restructure sections for style
- you do not “tighten” the narrative

You only make surgical changes that preserve the tuned prose:

- update one metric line
- replace one paragraph whose content is now stale
- add one short clarification where evidence changed

If a change cannot be applied surgically, that’s a signal that you need a new draft version, not an edit.

The point is to make “rewrite the article” an explicit decision, not an accidental outcome of a sync.

## How this interacts with publishing integrity

Locking isn’t only about tone.

It is also an evidence control.

When you lock a tuned draft, you are effectively saying: the proof point layer and the limits layer are complete. Any future change that touches those layers must be visible and deliberate.

That is exactly the posture you want when assistants are in the loop: evidence and constraints should be updated consciously, not polished away unconsciously.

## Limits (and what locking does not solve)

Locking is not a substitute for review.

- If the draft contains an incorrect claim, locking will preserve the incorrect claim.
- If the evidence changes materially, a surgical edit may not be enough; you may need a new version.
- If you lock too early, you’ll be forced into awkward micro-edits instead of letting the piece mature.

The goal is to lock at the point where further rewriting is more likely to delete value than add it.

## The lesson

If you let an assistant rewrite a tuned output, you’re not editing — you’re re-authoring.

Treat any tuned AI output like a fragile asset: lock it when it’s tuned, and only allow surgical updates thereafter.

