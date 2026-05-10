# Coverage Invariants: Stop Losing Requirements by Omission

Your assistant didn’t forget.

You just never mapped the new artifact, so it never got loaded again.

That is the quietest failure mode in AI-assisted workflows: drift by omission. Nothing breaks. Nobody complains. You simply stop using the best knowledge you have, because the workflow never points at it.

## Key takeaways

- In an AI workflow, an artifact that isn’t mapped might as well not exist.
- Drift by omission is worse than an obviously wrong source, because it fails silently.
- Enforce a coverage invariant: every source artifact must be assigned or explicitly excluded.

## Who this is for

This is for teams using AI assistants across long-lived repos and processes, where “what the assistant knows” depends on what you deliberately load and reference — not on what happens to be present in the filesystem.

## What an artifact is

In an AI-assisted workflow, an **artifact** is any structured markdown file the harness loads as an operational input: governance rules, implementation definitions, constraints, decisions, evidence. These are not reference material — they are the primary deliverables from which code is derived. The code is the by-product; the artifact is the thing worth protecting.

This matters for coverage because an unmapped artifact is not just a missing reference. In AI-assisted workflows, the artifact is the requirement. Losing it doesn’t remove a file from a filing system — it removes a constraint from the system. The code it governed keeps running as if the requirement never existed.

## The failure mode: you lose requirements without deleting them

In a normal codebase, unused code is a smell but it’s visible: there are dead functions, unused imports, or test gaps.

In an AI-assisted workflow, unused knowledge is easier to miss.

A new artifact is added:

- a design decision
- a lesson learned
- a runbook update
- a new constraint

But it isn’t referenced in any workflow plan, the index of sources, or the list of artifacts the assistant loads.

So the assistant never sees it again.

Months later, you look back and wonder why the write-ups feel generic, or why the same decision is being re-litigated. The knowledge exists. It just wasn’t connected to anything the workflow considers “loadable”.

That’s omission drift. And in AI-assisted workflows it is worse than it sounds: the artifact is the requirement. An unmapped artifact isn’t a missing reference — it’s a constraint that has silently stopped governing the system that was built from it.

## Why this is worse than “wrong artifacts”

A wrong artifact creates friction. Someone will eventually hit the mismatch and complain.

An omitted artifact creates false stability. The system keeps running and the output still looks plausible — it just gets subtly worse over time because it’s missing the newest constraints and the sharpest lessons.

In any AI-assisted workflow this shows up as:

- outputs that stop containing the best turning points
- decisions that no longer have their original rationale
- missing limits, because the limit lived in an artifact nobody loaded

## The decision: a coverage invariant

The fix is to treat coverage as a contract, not as a nice-to-have.

A coverage invariant is simple:

Every source artifact must be in one of two states:

1) Mapped to at least one workflow plan (it has an owner), or
2) Explicitly excluded, with a one-line rationale (so omission is a decision, not an accident).

There is no third state.

If a new artifact appears and it’s unmapped, the workflow is not “fine”. It is incomplete.

## The mechanism: inventory + mapping + enforcement

In practice, you need three parts:

### 1) An inventory

A deterministic inventory of source artifacts: what exists right now.

### 2) A mapping surface

A place where each article/pack lists the artifacts it draws from. The point is not to paste content. The point is to record ownership.

### 3) Enforcement

A check that reports unmapped artifacts as a gap, so you can’t miss them.

This does not have to be heavy. The goal is not bureaucracy. The goal is to prevent the “we didn’t realise that artifact existed” failure mode — for both humans and assistants.

## Keeping it cheap: pre-compute what changed

Coverage enforcement can be expensive if you constantly rescan and re-derive.

The pattern that keeps it cheap is the same one used elsewhere in well-governed assistant workflows:

- do the scan once
- store the result in a compact index
- have the assistant read the pre-computed output

That way you get strict coverage without making every session pay the full cost.

## Limits (and what this won’t do)

Coverage invariants don’t guarantee the assistant will use every artifact well.

- A mapped artifact can still be badly written.
- Mapping doesn’t replace good summaries and clear decisions.
- Some artifacts genuinely should be excluded (internal-only notes, transient working scratchpads). The invariant still holds — exclusion must be explicit.

The value is narrower and more reliable: you stop losing requirements by accident.

## The lesson

If an artifact isn’t mapped, it doesn’t exist — not to your assistant, and eventually not to you.

Make unmapped artifacts fail loudly, or you’ll drift by omission.

