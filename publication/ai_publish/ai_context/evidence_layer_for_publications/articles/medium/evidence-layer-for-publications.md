# Lint Your Claims: An Evidence Layer for AI‑Assisted Workflows

The easiest hallucination to ship isn’t a fake number.

It’s a real number with missing evidence.

If you use AI coding assistants to help draft articles, PRDs, or technical write‑ups, you will eventually hit this: the prose looks confident, the story is coherent, the metric sounds plausible — and nobody can point to the run artefact that proves it.

This is not a model failure. It is a workflow failure.

If you build with agent harnesses, this shows up in a specific way: your harness can be well‑governed, your code can be reviewed, and your operational outputs can still get “published” into drafts as confident claims with no replayable provenance.

## Key takeaways

- The easiest AI-workflow hallucination is a real metric that has lost its evidence trail.
- Treat any AI-assisted output as a truth surface: claims need anchors, not tone.
- An evidence audit doesn’t prove truth; it makes missing proof loud and fixable.

## Who this is for

This is for engineers who publish technical work from private systems (articles, internal PRDs, external write-ups) and want a workflow that makes it hard to ship “trust‑me” numbers after refactors and clean-ups.

## The failure mode: confidence without an artefact

Engineers are used to treating code as the thing that must be verifiable. Tests, linters, CI, deterministic builds. That mindset does not automatically extend to the outputs AI assistants help produce.

But any AI-assisted output that makes claims has its own truth surface:

- a throughput improvement
- a percentage auto‑accept rate
- a run time
- a distribution table
- a screenshot that supposedly came from a real run

If those claims drift away from artefacts, you don’t ship “a rough draft”. You ship fake confidence.

And fake confidence does not require fabrication. It requires losing the ability to prove what was once true.

## How it happened (a mundane case study)

In one publication workspace, we had drafts containing run‑derived metrics. They were real at the time.

Then we refactored the source system’s control plane and output layout (run IDs, manifests, where artefacts are written). During that work, outputs were wiped and reorganised. Nobody was trying to deceive anyone. We were just cleaning up.

The result was still an integrity failure: the publication repo could no longer *prove* some of its own numeric claims.

This is the publishing equivalent of “it worked on my machine”.

## Where this fits when you build with agent harnesses

When people talk about hallucinations in AI‑assisted engineering, they usually mean wrong code or wrong answers.

There is a quieter failure mode: wrong certainty.

You can prevent that by separating two concerns:

- **Runtime integrity:** guardrails that keep the system bounded while it runs (validation gates, deterministic checks, clear failure envelopes).
- **Output integrity:** guardrails that keep your written claims anchored after the run (evidence anchors, explicit gaps, and audits that flag “trust‑me” prose).

This article is about the second one.

## The insight: treat claims like code

If you can lint code, you can lint claims.

Not by asking an LLM to be honest, but by making honesty cheap:

1. Define a minimal evidence anchor for any numeric claim.
2. Make missing anchors noisy and reviewable.
3. Allow explicit exceptions when artefacts are genuinely missing, without pretending they exist.

The key is to stop relying on tone as a proxy for truth.

## The evidence layer: three states, no ambiguity

Every measurement-style claim in a draft must be in one of three states:

1) **Anchored** (publishable)

Include a run identity and an artefact pointer, for example:

- `run_id: <uuid>`
- a run manifest / run review artefact
- the export or screenshot provenance

This doesn’t require publishing private artefacts. It requires making the provenance locatable.

2) **Needs data** (not publishable yet)

Add a placeholder that describes the exact artefact to capture, for example:

NEEDS_DATA: run_id + artefact location + what to screenshot/export

This is a deliberate “do not make this up” marker.

3) **Evidence gap** (truth unknown in the current repo state)

When you believe the claim was real but the artefacts were wiped during refactors, mark it explicitly, for example:

EVIDENCE_GAP: artefacts wiped during refactor; rerun required; last observed YYYY-MM-DD

This turns “looks fabricated” into “known gap with a remediation path”.

## The mechanism: an audit that lints the prose

In this workflow, the evidence layer is enforced by a deterministic audit over drafts that produces a short review queue of unanchored claims.

The audit is intentionally simple: it flags numeric/measurement-style lines unless there is an evidence anchor nearby. It also catches obvious internal inconsistencies (for example, percentage breakdowns that sum to more than 100%).

It does not attempt to “judge” whether a claim is true.

It does something more useful: it tells you what the repo cannot currently prove.

## Why this matters more with AI assistants

AI assistants amplify the exact failure mode this pattern targets:

- They improve prose quality and coherence.
- They remove “draft smell”.
- They make unsupported claims look professionally written.

That’s what makes the failure dangerous. The writing looks finished before the evidence is anchored.

If you don’t have an evidence layer, your workflow quietly becomes: “ship the version that reads best”.

## What this pattern replaces

Without an evidence layer, teams usually oscillate between two bad extremes:

- **Paralysis:** refuse to write until every artefact is captured (progress dies).
- **Vibes:** write now, “we’ll add the proof later” (proof never arrives).

The evidence layer gives you a third option: write early, but make missing evidence explicit and auditable.

## The lesson

If your repo cannot point to the run artefact, your reader is consuming trust‑me prose.

Treat your AI-assisted outputs as a first‑class hallucination surface.

Lint the claims.
