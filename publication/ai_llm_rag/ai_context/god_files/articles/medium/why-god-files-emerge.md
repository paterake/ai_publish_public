# Why “God Files” Keep Emerging (Even When the Rules Are Clear)

If you’ve ever tried to keep a codebase clean while using AI coding assistants, you’ve probably seen this pattern:

You give clear guidance.

You state the architectural rule.

And a few sessions later, you still end up with a single file that contains everything: config parsing, orchestration, telemetry, error handling, formatting, and half a dozen unrelated helpers.

The file grows because each change is small and locally reasonable. The accumulation is what creates the mess.

This article is not about blaming the assistant. It’s about understanding the mechanism and designing a control that actually holds.

## What a “God File” Really Is

A god file is not “a large file”.

It is a file with many unrelated reasons to change.

Size correlates with the problem, but the actual failure is cohesion: the file becomes the default place to put anything “nearby”, and unrelated concerns accrete until the module is hard to reason about and harder to refactor.

## The Setup: Clear Rules, Clear Intent

Most teams start with a reasonable rule:

> One responsibility per file.

They might add a pattern:

- keep a stable façade as the public import surface
- move cohesive responsibilities into internal modules

On paper, this should prevent god files.

And yet they still appear.

## The Mechanism: Three Interlocking Causes

### 1) The nearest-open-file bias

When an assistant is asked to add a feature, it will usually modify the file that already contains related logic.

This is not stubbornness. It is the path of least resistance:

- the file is already open
- the surrounding code is already in context
- extending it satisfies the immediate ask

Creating a new file is a meta-decision about structure. Under task pressure, “extend the nearest file” is the default.

### 2) Task pressure wins the attention budget

Early in a session, architectural constraints carry weight because the assistant has just loaded them.

As the session fills with implementation detail, the relative weight of those constraints decreases. They are not ignored; they are outcompeted by the immediate signals of completing the task.

Even disciplined assistants tend to violate structural rules incrementally because no single step looks like a clear breach.

### 3) Prose rules have no failure signal

This is the root cause.

When a structural rule is violated, nothing fails.

There is no red build.

No test breaks.

No gate fires.

Without an unambiguous failure signal, the rule is advisory. Advisory rules erode under pressure.

## The Distinction That Matters: Preferences vs Constraints

An instruction written in a context document is a preference: a shape the assistant tries to satisfy when it can.

A gate is a constraint: a condition that stops the work when violated.

The key difference is that constraints do not live in the assistant’s context window. They live in the build. They cannot be “deprioritised” by task pressure.

If you want a structural rule to hold, encode it as a constraint.

## The Only Fix That Scales: Make the Structure Falsifiable

To prevent god files, you need a mechanism that makes the accretion visible and forces a decision.

There are many ways to do that. The shared pattern is:

1. Define the structural invariant (what must not happen)
2. Define a measurable proxy (how you detect it)
3. Fail fast when it drifts

The moment the rule has a failure signal, refactoring becomes the easier path:

- either split the responsibility
- or explicitly justify why the file is allowed to grow

Both outcomes are better than silent accretion.

## The Payoff

Once you have a structural gate in place, two things happen:

1. Assistants stop “accidentally” creating god files, because they can’t.
2. Humans stop policing structure by memory and taste, because the system makes the decision point explicit.

This is the broader lesson for AI-assisted development:

If a rule matters, don’t write it as prose and hope.

Give it a failure signal and let the system enforce it.
