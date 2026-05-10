# Your AI Assistant’s Memory Isn’t Backed Up

Most conversations about AI coding assistants start with the same threat model: can you trust what the assistant produces?

That matters. But if you use more than one assistant (Claude, Trae, Qwen), there is a quieter failure mode that will waste far more of your time: your rules don’t transfer.

You correct one assistant for weeks. It learns how you work. Then you switch assistants (or tools) and everything regresses. Not because the new assistant is “worse”, but because you stored the rules in a place only the first assistant could ever see.

This article gives you a simple mental model and a concrete fix: a three-layer memory architecture that makes multi-assistant work coherent, and makes your collaboration resilient to a new laptop, a re-install, or a tool switch.

## The Problem Nobody Warns You About

If you use a single assistant, you can get away with a flawed setup for a long time. You teach the assistant your preferences and it appears to “remember”.

As soon as you add a second assistant, the illusion breaks.

The rules you thought were “project rules” were actually stored as tool-specific memory:

- Update documentation in the same session as the code change
- Don’t run long application tests from the assistant; provide commands for a human to run
- Read the project workflow before making changes
- Keep publication tracking in one canonical place, not scattered across files

Those are not stylistic preferences. They are operating constraints that prevent drift.

If only one assistant can see them, you’ve built a collaboration that depends on a single client’s local state. That is a brittle architecture.

## The Mental Model: Three Layers of Memory

Most people talk about “AI memory” as one thing. In practice it is three layers with different properties:

| Layer | What belongs there | Persistence | Who can read it |
|------|---------------------|-------------|-----------------|
| Ephemeral | In-flight blockers, active run IDs, decisions still being evaluated | Ends with the session | One assistant |
| Assistant-local | Tool-specific preferences saved by one vendor | Vendor/tool-local; retention varies by product; not repo-backed | One assistant |
| Governing artifacts | Project rules, operating constraints, and workflow definitions stored in the repo | Git-backed | Any assistant |

The governing artifacts layer is the primary deliverable of AI-assisted workflows: structured human knowledge that cannot be regenerated without human effort — encoded constraints, decisions, and process definitions from which code is derived. Code is cheap to regenerate; these artifacts are not.

The mistake is putting stable rules in assistant-local memory.

Assistant-local memory feels “permanent” because it survives across sessions. But it is vendor/tool-local: no other assistant can read it. Retention varies by product — some vendors now cloud-sync this layer — but portability does not. And the backup policy is vendor-controlled, not yours: a product change or tier change can alter it without warning.

If you are building a system that relies on stable rules, those rules need to live somewhere durable and universal: the repository.

## A Second Dimension: What Kind of Knowledge?

The three-layer model tells you *where* memory lives and who can read it. A second dimension tells you *what kind of knowledge* you’re storing — and it lines up closely with how cognitive science breaks down long-term memory:

| Cognitive type | What it is | In a repo-based workflow |
|---|---|---|
| Working (short-term) | Current task context | The few files you load for this task + whatever is actively in flight |
| Semantic | Timeless facts and principles | Architecture notes, “why” decisions, standards and definitions |
| Episodic | Specific events tied to time and context | Run artefacts, incident notes, decision logs, experiments and outcomes |
| Procedural | “How-to” routines learned through practice | Repeatable workflows, automation, checklists, guardrails |

Seeing memory this way surfaces three practical design choices:

1. **Lean loading beats full recall.** Humans don’t pull every memory into working memory at once. Treat your project context the same way: load what the task needs, not everything “just in case”.
2. **Episodic memory should be queryable.** If an event matters later, store it as an artefact you can inspect and search, not as a vague recollection trapped in one tool.
3. **Boundaries should be enforced, not assumed.** The rule about where rules go needs to be part of the shared context itself, so every assistant learns the boundary before it accumulates new drift.

## The Two Symptoms (And Why They’re the Same Root Cause)

### Symptom 1: “This assistant keeps doing it wrong”

You switch from one tool to another and immediately see regressions:

- The assistant runs a long command you never wanted it to run
- It edits code but doesn’t update the associated docs
- It “finishes” a task but leaves the backlog in an inconsistent state

You correct it. Next time, you have to correct it again.

Most people diagnose this as assistant quality differences.

It usually isn’t.

It is memory architecture: one assistant learned rules that never entered shared context.

### Symptom 2: “I lost months of collaboration when I changed machines”

Even if you never change assistants, a machine change will force you to re-teach the same rules:

- Working style
- Definition of done
- What counts as publication-ready
- What to do and what not to do in this repo

Your code is in git. Your operating rules should be too.

## The Fix: A Shared “Working With This Human” Section

The fix is deliberately simple: create one shared, git-backed place for collaboration rules that any assistant can read before doing work.

Call it whatever fits your repo. The important part is the boundary: stable rules belong in the repo, not inside one assistant.

In this project, the shared context includes:

- Working style (what “done” means, how feedback is given, what to avoid)
- Behaviour rules (short, numbered, actionable)
- Current blockers that any new session must know

Then enforce a simple rule of thumb:

> If a rule should change assistant behaviour across sessions or tools, it belongs in shared context.

Everything else stays ephemeral.

## Preventing the Fix from Becoming Another Drift Vector

If you move rules into a shared file, the next failure mode is predictable: the file grows without discipline.

Within months, you can end up with a wall of “rules” that are really a list of edge cases. A new assistant can’t tell which constraints are load-bearing and which are historical noise. You’ve replaced “hidden memory” with “unreadable memory”.

The solution is not complexity. It is selectivity:

1. Does this change how we work on this project, or is it just what happened today?
2. Is this already covered by an existing rule, just applied to a new case?
3. Would I explain this to a new assistant starting tomorrow?

If you can’t answer “yes” to at least one of these, don’t promote it to shared context.

## Why This Matters

Multi-assistant workflows are becoming normal. Tooling will change. Models will change. Machines will be replaced.

The only durable surface you control is the repository.

If you treat shared context as part of your system design — not as an afterthought — you get three benefits immediately:

- Switching assistants stops being a reset
- New sessions start coherent without relying on chat history
- Your collaboration survives a new machine as cleanly as your code does

That is what this article is really about: not “better prompting”, but better architecture for collaboration.
