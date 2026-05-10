# Docs-Only Evidence: The Safest Default for AI-Assisted Workflows

If you need to read implementation code to produce your output, the output isn’t ready yet.

Not because the code is wrong.

Because producing output by code-reading pushes you into one of two failure modes:

- you leak private structure (paths, names, contracts, internal nouns), or
- you make claims that were never made explicit anywhere, and can’t be proven later.

This is why I keep the workflow intentionally code-blind by default: outputs are grounded in the artifact layer (README/PRD/governance docs), not inferred behaviour from implementation files. Publishing is the worked example here — the same boundary applies to any AI-assisted output that crosses from a private implementation into something external or credibility-bearing.

```text
OK: validation passed.
```

That output isn’t about style. It’s the point: output governance should be enforced by deterministic checks, not by trusting that an assistant “looked in the right place”.

## Key takeaways

- Producing output by code-reading is a leakage risk and an evidence risk, even when the code is correct.
- Docs-only evidence makes missing proof visible: it turns “inference” into a named gap you can close.
- Bounded inputs reduce both token burn and the chance of smuggling private structure into outputs.

## Who this is for

This is for you if you:

- use AI assistants to produce outputs from private implementations — articles, reports, technical write-ups, anything that crosses an output boundary
- care about credibility (what you can prove) and safety (what you must not leak)
- want a workflow that stays coherent across sessions and across different assistants

## The naive approach: “Just scan the implementation so the output is accurate”

If you’re serious about engineering, the instinct is understandable.

The code is the truth, so let the assistant read the code and summarise it.

In practice, this goes wrong in quiet ways:

1) The assistant starts naming the implementation because it has it in context.

You don’t notice at first. The output sounds more concrete. It uses specific nouns. It feels “real”.

Then you realise those nouns are exactly what you can’t expose: internal file paths, private module names, internal service names, and structural hints that make the write-up a map of your private system.

2) The assistant confidently infers behaviour that was never stated explicitly.

Code-reading encourages “I can see how it works, therefore I can claim it works this way”.

But any credibility-bearing output isn’t about what a reviewer could infer from source files. It’s about what you can state externally, prove, and maintain.

If a behaviour matters enough to surface, it needs an explicit evidence trail — not a one-off inference in a single session.

## The non-obvious risk: evidence drift, not code drift

Even when you avoid leakage, output-by-inference creates a second problem: the claims become hard to keep true.

A refactor can change behaviour or remove artefacts without changing what you already shipped.

The output survives. The evidence doesn’t.

That’s not a model hallucination. It’s a workflow bug: you produced something that wasn’t anchored to an evidence surface designed to survive change.

## The discipline: docs-only evidence by default

The alternative is simple, and it is intentionally strict:

- The only defensible claims in any AI-assisted output are the claims that exist in the source repo’s artifact layer.
- Outputs must not “discover truth” by scanning implementation files.
- Missing proof becomes explicit: a NEEDS_DATA placeholder that names the publish-safe evidence required.

This isn’t a purity rule. It’s an engineering boundary — and it follows directly from how AI-assisted development works.

In AI-assisted workflows, the **artifact** is the primary deliverable: the structured human knowledge (governance rules, implementation definitions, constraints, decisions) from which code is derived. Code is the repeatable, regenerable output. The artifact is what cannot be regenerated without human effort.

This inverts the traditional model. In traditional development, code is the deliverable and docs describe it. In AI-assisted development, the artifact is the deliverable and code is the by-product. Publishing works when you honour that inversion: publish what defines the system, not what was mechanically derived from it.

Code-reading bypasses this boundary. It treats the derived artifact as the truth source — which produces exactly the leakage and drift described above.

## The mechanism: treat docs as a contract, not as commentary

Docs-only evidence only works if the docs are treated as an evidence layer.

That changes how you write and maintain documentation:

- When a result matters, you record the conditions (what was run, under what constraints).
- When a limit exists, you state it plainly so it can’t be “smoothed out” by a later editing pass.
- When a claim can’t be made externally safe, you mark the gap instead of inventing a cleaner example.

The operational effect is powerful: you can review any output against a bounded set of sources, instead of re-deriving truth from code every time.

## The reinforcement: coverage invariants

A docs-only policy can still fail if the artifact set is incomplete or partially ignored.

So the governance layer adds a coverage contract: every source artifact must be mapped into at least one workflow plan, so nothing silently disappears.

When the invariant holds:

- new artifacts can’t slip in unnoticed
- article reviews can be grounded in the exact evidence list, not guessed
- assistants don’t have to rescan a repo to “remember what exists”

Docs-only evidence prevents leakage. Coverage invariants prevent amnesia. Together they protect the artifact layer that everything else is derived from.

## Limits (and when you should break this rule)

Docs-only evidence is a default, not a law.

- If you are writing internal docs for an engineering team with full repo access, code-reading may be appropriate.
- If you need one small implementation fragment to explain a decision, you can include it — but the fragment should be explicitly referenced and treated as evidence, not mined opportunistically.
- If your artifacts are stale or missing, the right fix is to improve the evidence layer, not to ship inferred behaviour.

The important thing is the direction of travel: reduce inference; increase explicit evidence.

## The lesson

“Truth by code-reading” is a governance smell in any AI-assisted workflow.

Produce outputs from explicit evidence, not inferred behaviour: artifact-layer only by default, and treat missing proof as a named gap you can close.
