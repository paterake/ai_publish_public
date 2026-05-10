# The Harness Is a Commodity

The community just figured out that the model isn’t the differentiator — the harness is.

Claude Code launches, and a clone appears within days.

That’s the moment the next conclusion becomes obvious:
the harness is a commodity too.

If a harness can be cloned quickly, the compounding investment isn’t “which tool”.
It’s the governance layer: the project-specific, git-backed rules, process discipline, and contracts that tell any harness how to behave.

This is not a thought experiment. I run the same governance layer across three different coding assistants today. When I swap tools, the behaviour transfers because it is anchored in the repo, not in any one assistant’s memory.

## Key takeaways

- The model sets the ceiling, the harness makes it usable, and the governance layer sets the floor.
- Harnesses proliferate quickly; optimising the harness is like optimising the IDE plugin. Useful, but not where advantage compounds.
- Governance is not “more prompting”. It’s a designed control surface: durable contracts, hard workflow triggers, and executable gates.

## Who this is for

If you have ever debated Claude Code vs claw-code vs Cursor vs Cline (or the next thing), this is for you.

Also: engineering leads deciding whether “AI tooling” is a procurement decision or an engineering capability you build.

## What you’ll learn

- Why “the harness is the differentiator” is correct, and still one layer short
- What the governance layer actually contains (and what it must not become)
- A simple test for whether your current setup will survive a tool swap

## The community’s half-insight

For a long time, the default debate was “which model?”:

- GPT-4 vs Claude vs Gemini
- bigger context windows
- “this one feels smarter”

Then the community had an uncomfortable experience: a better model didn’t fix brittle workflows.

Claude Code’s release surfaced the next layer:

> The model isn’t the differentiator. The harness is.

That is real progress. The harness matters because it is where the work becomes executable:
what context gets loaded, how tasks are scoped, what tools are available, how you apply safety boundaries, how you enforce a completion protocol.

But it is still not the durable advantage.

Because a harness can be copied.

If a clone can appear in days, “pick the best harness” becomes the same kind of mistake as “pick the best model”.
You are optimising a layer that is moving too quickly, and whose improvements will be commoditised.

## Why harnesses commoditise

Harnesses commoditise for the same reason that IDE extensions commoditise:

- features are legible (everyone can see what works)
- the primitives are shared (tool calling, file edits, diffs, code search, tests)
- the incentive is strong (a better harness turns a model into a product)

So the cycle becomes predictable:

1. A good harness ships.
2. The community copies the behaviour patterns.
3. The feature set converges.
4. The debate restarts at the next thin layer of differentiation.

This is not an argument against harnesses. You need one.

It is an argument about where to invest.

## The full insight: the governance layer

The durable advantage is the layer that the harness executes but cannot cheaply invent:

the governance layer.

The governance layer is the project’s “how we work” encoded in a form that survives:

- different sessions
- different assistants
- different machines
- different harnesses

It is not a single file of tips.
It is an artifact layer: structured human knowledge — governance rules, contracts, process definitions — encoded in a form that any harness can load and execute against. These are the primary artifacts of AI-assisted development; the code the harness produces is derived from them.

In practical terms, governance is what makes the same assistant behave consistently across a six-month codebase evolution, not just within a single chat.

If you swap the harness and your behaviour changes, you did not have governance.
You had familiarity.

## What governance contains (in practice)

If you want this to be real, governance needs three control surfaces.

### 1) Durable contracts

Every meaningful unit of work needs an entry point that states:

- what is in scope
- what must not change (invariants)
- how “done” is verified

This is the difference between “help me refactor this” and “refactor this module without changing the public interface; prove it by running X”.

Without contracts, the assistant is forced to infer intent from your chat style and recent context.
That works until it doesn’t.

### 2) Hard workflow triggers

Some actions must be non-optional.

If a certain type of change requires updating a boundary artifact, regenerating an index, or rerunning a validator, encode that as a trigger.

The test is simple:
if you have ever written “don’t forget to…” in a prompt, that is a workflow trigger you are currently holding in your head.

Humans forget. Assistants forget. Triggers don’t.

### 3) Executable gates

This is where governance becomes enforceable.

A rule without a failure signal is advisory.
Advisory rules erode under task pressure.

Gates are the mechanism that keeps behaviour stable when:

- the session is long
- the assistant changes
- a human is not reviewing every step

One concrete example: if a change crosses an interface boundary, the build fails unless the boundary contract is updated.

No memory required. No “please remember”. Just a system that refuses to accept a change that violated the contract.

## What governance is not

It is not “a better prompt”.

Prompts are session-scoped. They are useful. They are also ephemeral.

It is not a bigger CLAUDE.md full of preferences.

Preferences are not governance. Preferences don’t survive contact with urgency.

Governance is a designed system that makes the correct behaviour the default and makes incorrect behaviour noisy.

If your assistant behaves well only when you are watching closely, you do not have governance.
You have supervision.

## Ceiling vs floor: why governance beats model upgrades

Model quality sets the ceiling: how good the best answer can be.

Governance quality sets the floor: how reliably you get acceptable behaviour under real conditions.

Most teams try to raise the ceiling first.

But if the floor is low, raising the ceiling doesn’t help. You still get:

- “helpful” changes that violate invariants
- inconsistent refactors across sessions
- decisions that disappear when chat history disappears
- costs and latency that balloon under “research” behaviour

A governed weaker model can outperform an ungoverned stronger model on structured work because it is operating inside a system that constrains failure modes.

The harness makes the model usable.
The governance layer makes it dependable.

## The simple test

If you remember nothing else, use this:

> If you switched your primary AI coding assistant tomorrow — different tool, same project — how much of your setup transfers?

If the answer is “little or nothing”, your investment is trapped in the harness layer:
the tool, the UI, the muscle memory, the prompts that only one assistant sees.

If the answer is “nearly everything”, your investment is in governance:
the repo-backed contracts, triggers, and gates that any harness can execute.

That is where advantage compounds.

## Closing: stop optimising the tool

Harnesses matter. Choose one you can live with.

Then stop optimising the tool.

The tool is a commodity.

Invest in the governance layer: the project-specific, harness-agnostic content that makes any tool effective.

That is how you build AI capability that survives the next release cycle.
