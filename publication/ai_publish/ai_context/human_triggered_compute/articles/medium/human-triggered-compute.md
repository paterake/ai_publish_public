# Human‑Triggered Compute: The Delegation Boundary That Makes AI Workflows Safe

I asked the assistant what to publish next. It re-scanned 200 source files to answer.

A 37-line file already had the answer.

The issue wasn't capability — the assistant could scan those files, and the result was correct. The issue was that it shouldn't have decided to. The boundary between "what the assistant executes" and "what the human commands" had been left implicit, and an implicit boundary is no boundary at all.

## Key takeaways

- Some compute in an AI-assisted workflow should be human-triggered, not assistant-initiated: source indexing, coverage audits, build steps that rescan large directory trees. Publishing is the worked example here.
- The pattern: human runs the script; assistant reads the pre-computed result. Step 1 is a command. Step 2 is a read.
- This is a safety boundary first and a token efficiency gain second. When humans trigger compute, scope stays visible and controllable.

## Who this is for

This is for engineers and builders who use AI assistants to manage any multi-step workflow across sessions — publication pipelines, data pipelines, build workflows, documentation systems — and who have noticed that "what is the current state?" questions are unexpectedly expensive because the assistant is re-deriving state that already exists in a pre-computed file somewhere. The worked examples here are from a publishing workflow, but the pattern applies wherever pre-computed state exists.

## The failure mode: re-deriving pre-computed state

Most AI-assisted workflows have a natural information hierarchy:

- Raw source material (large: hundreds of markdown files, source docs, implementation notes)
- Pre-computed projections (small: a coverage audit result, a sorted queue, a generated priority table)
- Session actions (bounded: draft one article, update one PRD, apply one hygiene pass)

When the delegation boundary is implicit, the assistant will climb to the top of this hierarchy whenever it is uncertain. "What should I publish next?" becomes "let me re-read all the PRDs and re-derive the priority order" — because nothing told it not to, and it cannot tell the pre-computed file exists without reading it first.

The result is expensive for the obvious reason (token cost, session latency), but also for a less obvious one: scope becomes invisible. You cannot see what got rescanned. You cannot control it without interrupting the session.

## The decision: human-triggered compute

The structural decision that prevents it is a clean separation of responsibilities:

- **Human triggers.** Source indexing, coverage audits, build steps that produce pre-computed outputs. These are commands. The human decides when and what scope.
- **Assistant reads.** Pre-computed JSON, sorted queues, audit results. These are projections. The assistant reads them; it does not regenerate them.

This is stated explicitly in the workflow documentation:

> *"I do not trigger this. The human commands it. The JSON is the durable output I read."*

That sentence in the workflow doc is a delegation contract. The human triggers the rescan. The assistant reads the snapshot output. Neither steps into the other's role.

## The mechanism: pre-computed outputs as the delegation surface

Three artefacts illustrate the pattern in the publishing workflow.

**An index rebuild** scans the sources and writes a durable snapshot. It records what is new, what changed since the last run, and what is currently unmapped. This is expensive compute — it walks the full source directory. It is a human command, not an assistant action.

**A coverage audit** reads the snapshot. It does not rescan sources. It reads the pre-computed fields and produces a compact audit result (under 20 lines when clean). The assistant can run this safely because the scope is already bounded by the previous human-triggered step.

**A priority table** is the pre-computed answer to "what should I do next?" The assistant reads it at session start. It does not re-derive the priority order. The computation already happened; reading the result is the session action.

## The safety benefit

When the human triggers compute, two things become visible:

**What got rescanned.** The human ran a scoped rescan. That command names the scope. The assistant cannot accidentally rescan something else because it did not trigger the scan.

**When it happened.** The snapshot output has a timestamp. If the assistant reads a snapshot that is three days old, that is visible — and the decision to refresh or not belongs to the human.

This is not about distrusting the assistant. It is about making scope an explicit human decision rather than an implicit assistant inference.

## The token efficiency benefit

The efficiency gain is real but secondary.

Reading a generated table is cheaper than re-deriving the order from a pile of prose documents. Reading a snapshot index is cheaper than rescanning hundreds of source files.

But efficiency alone is not a sufficient reason for the boundary. An assistant that re-derives everything on demand is slow and expensive; one that does so while accidentally crossing project scope boundaries is also incorrect. The safety benefit and the efficiency benefit point to the same structural decision.

## Limits (and what the boundary does not solve)

Human-triggered compute bounds scope. It does not prevent:

- **Stale pre-computed outputs.** If the human has not run the rescan recently, the snapshot may be out of date. The assistant reads what is there. This is intentional — the human controls refresh timing — but it means the assistant's answers are only as current as the last human-triggered sync.
- **Scope errors in the script itself.** If the rescan is scoped incorrectly, the snapshot is wrong. The human triggered the error; the assistant faithfully reads it.
- **Boundary erosion over time.** As assistants become more capable, the temptation to let them trigger compute grows. The boundary requires active maintenance — not just as a workflow rule, but as a documented contract in the skill or workflow file.

## The lesson

The delegation boundary isn't about what the assistant *can* do.

It is about what should require an explicit human command.

When compute is human-triggered, scope becomes visible: the human decided what got rescanned, at what time, with what parameters. The assistant reads the result. Neither can accidentally become the other.

That is not a token optimisation. It is a safety posture — and the token savings follow from it.
