# Analogies Are Enabling Infrastructure

*Einstein didn’t use thought experiments to reach more people. He used them to think more precisely.*

I learned that lesson the hard way while building an agent harness: an orchestration loop that routes to tools and sources, synthesises drafts, and uses gates to decide whether to keep searching or stop.

The harness could write fluent, plausible explanations and plans on demand. It could also do the wrong thing with complete confidence: optimise for wording when the real problem was missing evidence; stop early when it hadn’t actually gathered anything; “helpfully” invent structure that wasn’t in the source.

What changed the trajectory wasn’t another page of instructions.
It was a handful of analogies that conveyed intent.

Not analogies as decoration.
Analogies as infrastructure: a shared vocabulary that compresses the point of a mechanism into something an agent (and a human) can reliably act on.

Most people treat analogies as a soft skill.

You add an analogy when you need to explain something difficult to someone “less technical”.

That framing is backwards.

A good analogy is not a simplification. It’s an extraction of essence: a way of expressing the structural truth of a mechanism without dragging the reader through implementation detail.

That makes analogies a form of infrastructure.

Infrastructure is not glamorous, but it changes everything because it scales. A good analogy propagates through every explanation that touches the concept — across documents, across teams, and now across AI assistants that will explain your work when you are not in the room.

---

## Key takeaways

- Analogies are a precision tool, not a “dumbing down” tool.
- A canonical analogy vocabulary reduces drift across docs and across agent runs.
- Analogies can carry intent: they tell an agent what to optimise for, and what to treat as failure.
- A fluent but wrong analogy is worse than jargon; treat analogies as something you verify, not improvise.

---

## Einstein’s mistake, and why it wasn’t a mistake

Einstein “rode a beam of light”.

That wasn’t a popularisation for non-physicists. It was how he reasoned about relativity above the level of the maths. Translating an abstract mechanism into a human-grounded thought experiment didn’t just make the idea accessible. It forced precision of understanding that improved the physics itself.

That is the first principle:

If you can’t explain a concept in human terms, you probably don’t understand it deeply enough to build it reliably.

Not because jargon is bad — but because jargon can hide gaps in understanding.

---

## Analogies as intent (not explanation)

When you work with an agent harness, you’re not only writing for readers. You’re writing for a system that will make decisions when you’re not present: which sources to trust, when to keep searching, whether an answer is “good enough”, whether to prefer determinism over fluency.

Analogies helped because they act like interfaces:

- stable names for recurring concepts
- explicit boundaries (what it covers, what it does not)
- a default mental model that survives across sessions, tools, and assistants

In practice, we used analogies as a way to convey intent in a single sentence that kept working long after the immediate conversation ended.

---

## The casual browser is the multiplier

There is a second principle that matters even more for public work.

Not everyone who encounters an idea is looking for it. Some people arrive by accident: a colleague shares a post, a recommendation lands at the right moment, a conversation triggers curiosity.

Those are the people who carry ideas furthest.

They are not going to run your code or read your architecture diagrams. They are going to take one human-grounded insight and apply it in a different domain. That is how understanding propagates beyond the original implementation.

The practical implication is uncomfortable:

Writing only for experts is writing for the smallest possible audience.

Experts will still find depth if it exists. The casual browser needs an entry point they can carry away.

That entry point is usually an analogy.

---

## What we actually did with them

We treated analogies as first-class artefacts, not throwaway lines.

1. Picked a small set of canonical analogies and wrote them down in one place.
2. For each analogy, stated where it holds and where it breaks.
3. Used the analogy term deliberately in PRDs, task briefs, and reviews as a shorthand for intent.
4. Checked that the analogy assigned responsibility correctly and didn’t hide the primary failure mode.

The goal wasn’t “make it easier to understand”.
The goal was “make it harder to build or accept the wrong system”.

---

## A few examples that are doing real work

These aren’t “cute metaphors”. They’re decision tools.

### The tool orchestrator as the librarian

The librarian doesn’t store the books. The librarian makes the books findable.

That single sentence corrects a common category error: assuming the content producer decides how content is retrieved. In a library, the librarian decides. In a tool-orchestrated system, the orchestrator decides — dynamically, based on tool definitions.

The analogy doesn’t replace the technical definition.
It changes who you think owns the responsibility.

### Retrieval-Augmented Generation (RAG) as an open-book exam

A large language model (LLM) without retrieval is sitting a closed-book exam: it answers from memory.

RAG is an open-book exam: you retrieve notes and then synthesise an answer.

That analogy exposes the real failure modes:

- bad notes (retrieval) produce confident wrong answers
- good notes with poor synthesis waste evidence

It also makes a governance stance obvious:
you don’t accept an answer without evidence just because it sounds fluent.

### A quality gate as an editor

Before an article is published, an editor checks it against a rubric: no vague claims, proper citations, minimum substance.

A quality gate does the same thing for model outputs, except it is stricter. It can’t “request a rephrase”. It can only reject and trigger more evidence gathering.

That one constraint — the gate is an evidence check, not a style check — changes how you design the whole loop.

The analogy makes it harder to build the wrong system.

It also conveys intent to an agent cleanly:
if the editor rejects, you don’t “write nicer”. You go back to research.

### The agentic loop as a detective

A detective doesn’t stop at the first lead. They gather evidence, notice what’s missing, and decide whether they can close the case.

That frame helped in two ways:

- It made iteration feel like the default, not a failure.
- It made premature certainty feel like a bug, not a personality trait.

We then turned the analogy into operational constraints: bounded iteration, explicit actions (“search again”, “scan for an exact name”, “stop”), and guardrails against early exit.

---

## The risk: fluent misinformation scales faster than jargon

Infrastructure can fail.

The failure mode of analogies is not that they are “too simple”.

The failure mode is that they are memorable and wrong.

A fluent but inaccurate analogy spreads faster than accurate jargon because it is easier to repeat.

This is why analogies need a verification posture:

- verify the analogy against the real mechanism
- state where it holds and where it breaks
- avoid inventing new analogies casually because they “sound right”

If you can’t verify it, use the precise technical language until you can.

---

## A practical rule

If you want a simple operational habit:

When you introduce a new core concept, do not ship the explanation until you can express it in one human-grounded analogy that passes two tests:

1. It assigns responsibility correctly.
2. It doesn’t hide the primary failure mode.

If it fails either test, it’s not infrastructure. It’s decoration.

---

## Close

Analogies are not a way of making hard work look easy.

They are a way of making hard work *thinkable* — above the level of the code — so the idea can propagate without losing its structural truth.

That is infrastructure.
