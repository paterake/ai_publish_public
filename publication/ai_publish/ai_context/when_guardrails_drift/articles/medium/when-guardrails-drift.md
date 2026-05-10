# When the Guardrails Drift: How to Engineer a Self-Maintaining AI Harness

I built a governance harness to stop AI coding assistants from drifting.

Then I found the harness itself drifting — the same LinkedIn formatting rules living in four different documents, each slightly different.

The fix wasn’t “be more careful”. It was applying the same engineering discipline to the harness that I apply to the systems the harness governs: one canonical source per rule, pointers everywhere else, and a mechanical check that fails loudly when drift reappears.

Here’s the concrete artifact that convinced me this needed to be engineered, not wished for:

```text
14 files checked — all clean.
```

That one line is the point: I don’t want correctness to depend on whether an assistant happened to read the right file in the right order in the right session.

## Key takeaways

- If a rule is copied into multiple files, it will drift — even when each copy is “helpful”.
- “Self-maintaining” harnesses depend on behaviour; “self-healing” harnesses use mechanical checks.
- Treat harness documents like code: one owner per rule, pointers everywhere else, and a failing pre-flight.

## Who this is for

This is for you if:

- you’re using an AI coding assistant (or several) and you’re trying to keep work coherent across sessions
- you already have guardrails, but you’re starting to see subtle inconsistencies creep in
- you’ve felt the cost of re-deriving “how we do things” because the canonical source wasn’t actually canonical

## The moment I realised the harness was drifting

The failure wasn’t a bad patch.

It was something quieter: documentation drift inside the governance layer.

I found that the LinkedIn posting rules (the sort of rules that decide whether a draft is copy/paste-ready) existed in four places:

- the canonical writing standards document
- the workflow document that describes the end-to-end publishing process
- the assistant guide that onboards a new session
- two skill runbooks that were meant to be orchestration only

Each copy looked innocuous. Each was written with good intentions. And each had drifted slightly.

This is the governance version of configuration drift: the system still “works”, but you no longer know which version you are actually running.

With one wrong rule, you notice the problem. With four plausible copies, every assistant believes it is following the rules — and none of them are guaranteed to match.

## Why this happens (even in “well-governed” setups)

This drift pattern is almost inevitable unless you design against it.

The incentives are aligned for duplication:

- A skill file wants to be helpful, so it restates rules as reminders.
- A workflow file wants to be complete, so it restates the same rules “so you don’t have to click”.
- An onboarding guide wants to be safe, so it repeats the key bits.

Nobody deletes the duplicates because they look useful.

But “useful” is exactly what makes them dangerous. They’re close enough to the truth that they’re trusted, and far enough from the truth that they become a slow source of errors.

In a multi-assistant workflow, this compounds: one assistant updates the canonical standards, another updates the workflow, a third edits a skill, and you now have a governance fork without noticing.

## The pattern: pointer-not-copy architecture

The fix is boring, which is the point.

For any rule that governs publishing behaviour (format, safety, voice, proof point discipline), pick one canonical home and make everything else a pointer.

That means:

- The canonical document contains the full rule.
- Every other document contains one line that points to the canonical source.
- If a document needs to provide context, it explains why the rule exists (a consequence), not the rule itself.

This is the same separation you’d use in software:

- Implementation: the actual rule, with details.
- Interfaces: references to the rule, with responsibility boundaries.

Once you do this, updates stop being a synchronisation task. You don’t need to remember to edit four copies. There aren’t four copies.

## Self-maintaining vs self-healing (the non-obvious consequence)

At this point, it’s tempting to say: “Great — we’ll write a policy that says everything must point to the canonical sources.”

That helps, but it’s still a behavioural control. It’s self-maintaining, not self-healing.

Here’s the distinction:

- A self-maintaining harness depends on everyone (including the assistant) following the process: read the right files, don’t duplicate, keep standards in sync.
- A self-healing harness has at least one mechanical check that runs regardless of memory, attention, or goodwill — and fails loudly when drift appears.

Behavioural controls are necessary. They are not sufficient.

The thing I want to be true is: even if a future assistant reintroduces a duplicated rule because it “felt helpful”, the repo fails fast and forces the correction before that drift ships into a published draft.

That’s what a mechanical check buys you.

## The mechanical check: “fail the session if drift is present”

The simplest useful enforcement layer is an audit script that looks for known drift patterns:

- a skill file restating LinkedIn rules (instead of pointing to the canonical standards)
- the onboarding guide containing format rules in-line (instead of pointing to the canonical standards)
- the workflow document re-listing format blocks (instead of pointing to the canonical standards)

This is intentionally narrow. It doesn’t try to be a general-purpose linter. It checks the specific “drift vectors” that matter to the harness contract.

The output is binary: clean or fail.

```text
14 files checked — all clean.
```

The second the script prints “FAIL”, you have an engineering problem you can fix — not a social problem you have to remember.

## The second drift I didn’t expect: undocumented structure

The same session exposed a second class of governance drift, and this one surprised me because it wasn’t about rules.

It was about structure.

There was a two-level publishing backlog hierarchy:

- a project-level operational tracker (the entry point you start each session from)
- module-level checklists that exist only when an article needs evidence capture

That hierarchy wasn’t documented in the assistant guide.

So a cold session had to rediscover it by scanning the publication directory, opening files, and inferring “how this repo works” from the filesystem.

That’s exactly the behaviour good harness design is meant to avoid.

The whole point of a governance layer is to make the collaboration contract loadable, not rediscovered.

And the fix was almost embarrassingly small: a short addition to the assistant guide that explains the two levels, the navigation pattern, and where “missing proof point” work belongs.

Again: boring is the point. The harness drift wasn’t a deep design flaw. It was a missing invariant and no mechanical enforcement.

## How to make a harness resilient to its own drift

This is the checklist I now treat as non-negotiable for harness maintenance:

1. Pick one canonical home per rule category (format, voice, safety, workflow).
2. Replace every duplication with a one-line pointer.
3. Add a propagation rule: if a canonical file changes, dependent docs must be checked in the same session.
4. Write a narrow audit that checks for known drift patterns and fails loud.
5. Treat undocumented structure as drift too: if a new assistant has to scan to rediscover “how to operate”, the harness is missing an entry point.

## Limits (and when this won’t save you)

This isn’t a replacement for review, and it won’t catch everything.

- A drift audit can only catch what you encode. It’s most effective for recurring failure patterns, not novel mistakes.
- Over-enforcement can create friction. Keep the checks narrow and tied to real failure modes you’ve observed.
- You still need one human decision-maker. Canonical source discipline only works if someone owns each canonical file and treats it as a contract.

But within those limits, this is a high-leverage move: it converts governance drift from “quiet divergence over time” into “one loud failure you can fix immediately”.

## The lesson

A governance harness that can’t detect its own drift isn’t fully engineered.

Apply the same canonical-source discipline to the harness files as you would to any other part of the system: one owner per rule, pointers everywhere else, and a mechanical check to verify.

What’s the one drift pattern you’ve already seen in your own assistant workflow — and what would a failing check look like for it?

