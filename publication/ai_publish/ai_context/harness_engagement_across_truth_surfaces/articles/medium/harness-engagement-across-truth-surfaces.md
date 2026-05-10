# Two Truth Surfaces: How to Engage an Agent Harness for Implementation and for Output

People talk about “hallucinations” as if they are a model defect.

In practice, most hallucinations that matter are governance defects: a system produced something plausible, and nothing in the workflow made it expensive to be wrong.

Agent harnesses are a response to that problem. But there is a trap: the same harness can be used to govern two different kinds of truth, and they fail differently.

If you engage the harness the same way in both contexts, you will ship the wrong failure mode.

Here is a small example of why this matters. I asked for guidance on whether two publication ideas were distinct. The workflow split them into two drafts with different proof requirements. Then a contradictory argument arrived: “this looks like spamming; keep it to one”. The assistant collapsed the work into one draft as a publishing decision. When challenged on the inconsistency, the decision was reversed and a new rule was added: corpus-level publishing decisions must be explicit and must be logged with a rationale.

That is what harness governance looks like when it is working: not perfection, but fast detection, correction, and tightening the rules so the same class of mistake becomes harder to repeat.

## Key takeaways

- Your harness governs more than code: output claims are a separate truth surface.
- Implementation truth is enforced with diffs and checks; output truth is enforced with evidence anchors.
- If you reuse the wrong controls, you’ll ship the wrong failure mode (polished, unprovable confidence).

## Who this is for

This is for teams using AI assistants with any kind of harness/guardrails, who have solid code governance but still struggle to keep published claims (numbers, tables, screenshots) anchored to replayable evidence.

## The common mechanism, two different jobs

Across this work, agent harness engagement is the common mechanism:

- In the implementation repository, the harness exists to prevent wrong code, silent drift, and unbounded changes across sessions.
- At the output boundary — publishing in this example, but any output that carries credibility claims — the harness exists to prevent wrong confidence: polished prose and tidy metrics that cannot be replayed to artefacts.

Same mechanism. Different truth surfaces.

## Truth surface 1: implementation truth

When you are governing code, your truth surface is concrete:

- the diff
- the tests
- deterministic validators (format, structure, forbidden references)
- reproducible outputs

The harness should be engaged to keep changes bounded and checkable:

- prefer small diffs with clear intent
- run deterministic checks after edits
- treat failing tests as a hard stop, not a “maybe it’s fine”

The failure mode here is obvious: broken builds, failing tests, runtime exceptions, regressions.

But code has a more subtle failure too: changes that “look right” and read well, but shift behaviour in ways nobody notices until later. That is drift. Harness governance is how you make drift detectable rather than inevitable.

## Truth surface 2: output truth

When you are governing outputs — anything that carries claims beyond the implementation boundary — your truth surface is different.

Recipients cannot run your test suite. They cannot inspect your diffs. They are consuming claims.

So the integrity unit is not “the change” but “the claim”:

- a throughput improvement
- a percentage
- a run time
- a table of outcomes
- a screenshot that supposedly came from a real run

The failure mode here is not broken code.

It is wrong confidence: a coherent narrative that cannot be proven from artefacts.

This is especially easy to ship when AI assistants are involved, because they remove draft smell. They turn incomplete work into prose that sounds finished.

## The engagement mistake: using the wrong controls

The common mistake is to use implementation-truth controls to try to secure output truth.

For example:

- “The code exists, so the claim is probably true.”
- “The system seems reasonable, so the numbers are plausible.”
- “The assistant wrote it confidently, so it must have come from a run.”

That mindset produces output drift even when your implementation is governed.

Refactors make the problem worse: outputs are reorganised, run artefacts are wiped, and the draft keeps the numbers. Nothing malicious happened. The repo just lost the ability to prove its own claims.

## A practical harness engagement playbook

If you want to prevent hallucination and fake confidence across both surfaces, engage the harness differently.

### For code work

- Demand small, testable diffs.
- Prefer deterministic validators and contract checks over narrative assurances.
- Treat “should work” as a failure mode; prove it by running checks.

### For output work

- Treat every measurement-style statement as a lint target.
- Require an evidence anchor for numeric claims: a run identity plus an artefact pointer (manifest, export, screenshot provenance).
- If evidence is missing, mark it explicitly as a gap and route it into a review queue.

Publishing is the worked example here. The same controls apply to any output that carries credibility claims — reports, dashboards, technical write-ups, anything where the recipient is consuming claims rather than inspecting diffs.

## The lesson

An agent harness does not “make the assistant honest”.

It makes honesty cheap by enforcing the truth surface you care about.

Match the harness engagement to the surface:

- tests and validators for implementation truth
- evidence anchors and audits for output truth

Do that, and “hallucinations” stop being a mysterious model quirk.

They become what they always were: a failure you designed your workflow to allow.
