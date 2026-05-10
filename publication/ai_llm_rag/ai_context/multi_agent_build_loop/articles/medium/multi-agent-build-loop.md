# The Multi-Agent Build Loop: From Intent to Success

Success wasn’t a feeling. It was a run you could defend.

That’s the inversion that made AI-assisted development work on a real system: stop arguing about whether the output *looks* plausible, and start insisting on evidence you can point to.

Most “multi-agent” write-ups start with a cast list: planner, coder, tester, reviewer. The version that mattered here wasn’t a cast list. It was a loop that repeatedly turns intent into proof.

Here’s the loop in one line:

**Human intent → contract (PRD) → bounded execution (thin runners) → evidence (run_id artefacts + telemetry) → judgement (review) → success.**

Once you have that, iteration becomes fast without becoming fragile.

---

## What This Loop Built (In Practice)

This isn’t an abstract workflow. It’s the build process that produced the full implementation stack behind the publication series:

- A classification framework for large-scale, taxonomy-bound labelling
- An ingestion substrate that turns messy documents into a queryable corpus
- Hybrid retrieval that combines retrieval strategies instead of betting on one
- Evidence shaping and query orchestration that turns “a question” into “inspectable inputs”
- Agentic retrieval that is bounded, auditable, and reviewable by run artefacts
- A knowledge-graph surface for traversal and retrieval beyond chunk search
- Two query surfaces (direct API/UI and tool-style calls) without duplicating logic
- An HTTP agent surface that turns questions into multi-step tool calls
- Core LLMOps primitives: run context, telemetry, run review artefacts, and drift containment
- Visualisation and exploration workflows that treat “chart generation” as a governed pipeline

The point of listing these is not to show breadth. It’s to make the claim precise: this loop wasn’t used to build a single feature. It was the delivery mechanism for an entire system.

---

## The Real Multi-Agent Split: Coordinator/Judge vs Executor

When people say “multi-agent”, they often mean “multiple LLMs talking to each other”.

In practice, the split that works is simpler and more operational:

- **Coordinator + judge (architect/lead developer):** the harness and the higher-level reasoning that plans, routes tasks, reads constraints, and decides what counts as success.
- **Executor (junior developer):** a local model (or a narrower assistant context) that does the concrete work: implementing a function, rewriting a draft, applying a refactor, running a bounded command, producing a structured output.

That split is only meaningful if the coordinator can *verify* what the executor did without relying on vibes.

So the centre of gravity becomes: **evidence**.

The practical detail that made this work is that multiple assistants behaved consistently. The coordinator role wasn’t “one model”. It was a governed harness that made different assistants interchangeable:

- one assistant could review a run’s AIOps artefacts and spot gaps
- another could implement the fix
- a third could sanity-check the change against the contract

Because they were operating under the same rules and the same review posture, the loop held together across tools and sessions.

Concretely, the coordinator/judge role was carried by a shared harness applied across three different coding assistants:

- **Claude Code** (repo-scale planning, review, and refactors)
- **Trae** (IDE-integrated execution: edits, iteration, and structured task-running)
- **Qwen** (local/offline assistant when needed; also the local runtime model used to execute the pipelines)

The point is not that any one of these is “the” agent. The point is that the governance made them behave consistently enough that they could be swapped in and out of the loop without changing the standard of evidence.

---

## Intent Needs A Shape: The PRD As Contract

The fastest way to get an assistant to do something is to describe the outcome conversationally.

The fastest way to get a system to drift is to keep doing that.

The fix is to make intent durable:

- What is being built, in one sentence?
- What does “done” mean in observable terms?
- What is explicitly out of scope?
- What evidence proves the claim?
- What failure modes are acceptable (degrade) vs unacceptable (fail)?

I treat that as a contract: not as paperwork, but as the *coordination mechanism* between the human and the assistants.

Once intent is a contract, it can be checked for alignment later. Without it, every new session becomes a re-interpretation of the goal.

---

## Bounded Execution Is What Turns “Agentic” Into “Deliverable”

Assistants can write code quickly. They can also create infinite work quickly:

- “Let’s just try another approach.”
- “Let’s add a new abstraction.”
- “Let’s run again with different parameters.”

If you let that behaviour leak into pipelines, you don’t get autonomy. You get unboundedness.

The solution is to keep execution behind thin entry points:

- A small number of commands/runners that are treated as public interfaces
- Explicit bounds: max iterations, max tool calls, max retries, timeouts
- Explicit budgets for LLM calls and output sizes
- Explicit degrade modes (what happens when the system can’t complete the full plan)

The point is not to slow the assistant down. It’s to make “run the thing” repeatable, comparable, and safe-by-default.

Without thin entry points, you can’t reliably answer: “What did we just run?” or “Is this behaviour stable?” You just have a pile of changes and a feeling.

---

## AIOps As The Judge’s Eyes

I use “AIOps” here in the most practical sense:

**signals that make outcome correctness inspectable.**

Not dashboards for their own sake. Signals that answer concrete questions:

- Did we stay within budget?
- Did we hit a known degrade path?
- Did we silently drop data?
- Did retrieval quality regress?
- Did the agent loop terminate for the right reason?

For this loop to work, “the run” needs to produce:

- **Per-run artefacts** (outputs, manifests, summaries)
- **Structured telemetry** (events with consistent fields)
- **A run review** (a scorecard that can be read quickly)

That isn’t glamorous work. It’s the work that turns a local experiment into something you can trust and evolve.

This is also where the loop becomes concrete: assistants review the run evidence, identify a gap or failure mode, make a targeted code change, re-run locally, and repeat until the outcome is defensible.

---

## The Stitching Key: Why Everything Hangs Off A run_id

The crucial mechanism is a correlation ID: a `run_id`.

If you don’t have one, you can’t join the story up:

- Which code changes produced these outputs?
- Which outputs correspond to which evaluation?
- Which telemetry belongs to which run?
- Which warnings were new vs already known?

A `run_id` is how you turn “we ran it” into an auditable claim.

In this workflow, the `run_id` is not an implementation detail. It is the bridge between:

- the PRD’s claims,
- the system’s behaviour,
- and the next engineering decision.

It also prevents a specific class of failure: **fresh IDs generated mid-run**.

If IDs are re-generated inside the call chain, you get fragments: logs with one ID, artefacts with another, telemetry that can’t be joined. The run becomes unreviewable. You can still ship it. You just can’t *manage* it.

So the invariant is simple: **entry points own the lifecycle**. Generate or accept the `run_id` once, then propagate it everywhere.

---

## How The Loop Plays Out In Practice

Here’s what a real iteration looks like when you take the loop seriously.

### 1) Start with intent, not code

The human intent is framed as a contract:

- “The retriever must never enter an unbounded reformulation loop.”
- “Every LLM call must emit structured telemetry with the same `run_id`.”
- “If context budget is exceeded, degrade deterministically rather than truncating silently.”

That contract defines what you will measure and where you will look when it fails.

### 2) Delegate to the executor with tight constraints

The coordinator (harness + assistant) delegates a bounded task:

- implement a specific guardrail
- wire the telemetry emission at a defined boundary
- add a deterministic stop condition

The executor’s job is not to “make it better”. It’s to implement a bounded change that can be evaluated.

### 3) Run through the thin entry point

Instead of “try it in a notebook”, you run the system through the same small surface every time.

That yields comparable artefacts:

- outputs in a per-run folder
- manifests describing what was executed
- telemetry events that can be queried

### 4) Review by run_id, not by opinion

The review posture is intentionally boring:

- load the run review scorecard first
- scan the delta vs the previous run (if available)
- only dive into raw telemetry when something is unexplained

This is where the coordinator acts as judge.

If the run fails, the result isn’t “the assistant got it wrong”. The result is a concrete diagnosis:

- we violated a bound
- we degraded without declaring it
- we emitted a warning without an evidence trail
- we regressed a metric that must be gated

### 5) Update the contract or the system — explicitly

This step is where most AI-assisted projects drift.

When something fails, there are only a few valid moves:

- **tighten the system:** add a bound, add a gate, add a degrade mode
- **tighten the evidence:** improve telemetry, run review, evaluation harness
- **tighten the contract:** change the PRD claim, or reduce scope honestly

What you don’t do is “ship anyway and hope the next run is better”. That’s how silent failures become your baseline.

---

## Why This Counts As Multi-Agent (Even If Nothing Is “Chatting”)

The loop is multi-agent because it has multiple autonomous actors with different responsibilities:

- the human sets intent and accepts trade-offs
- the coordinator plans and judges against evidence
- the executor produces concrete changes
- the system emits signals that constrain what can be claimed

If you remove any of those pieces, you still have “AI-assisted coding”. You don’t have a multi-agent delivery loop.

The reason this matters is not taxonomy. It’s failure containment.

This loop prevents two common failure modes:

1) **The assistant becomes the author of the goal.** You stop steering, and “helpful” becomes “direction”.
2) **Correctness becomes aesthetic.** You decide quality by how plausible the output looks, rather than by whether it meets the contract with evidence.

---

## The Non-Obvious Benefit: The Loop Survives Context Loss

Assistant sessions are short. Context windows are limited. Even with good tooling, you will lose continuity.

The loop survives because the important state lives outside the conversation:

- the contract (intent) is durable
- the entry points are stable
- the evidence is keyed and stored
- the review posture is repeatable

This is the closest thing I’ve found to “scaling” AI-assisted development without pretending the model has long-term memory.

---

## Close: Stop Treating The Build As A Conversation

If you want the multi-agent story to be real, treat it as engineering:

- split roles
- bind execution behind thin surfaces
- key evidence to a run identifier
- review by artefacts, not by confidence

That doesn’t remove creativity. It removes the most expensive kind of uncertainty: the kind you discover a week later when you can’t reproduce what “worked”.
