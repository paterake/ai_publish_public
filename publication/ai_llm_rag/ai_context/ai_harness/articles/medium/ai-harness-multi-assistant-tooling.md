# Your AI Coding Assistant Rules Are Invisible to All Your Other AI Coding Assistants

*How a shared harness keeps three AI tools in sync from a single source of truth — without a synchronisation step*

Three AI coding assistants. Three separate tool config directories. Three different sets of rules — each growing independently, none visible to the others.

This is what the standard setup produces the moment you add a second AI tool to a codebase already configured for one. A rule written for one assistant lives in that assistant’s tool-specific folder. The other tools do not see it. A hook written for another tool does not run when the first assistant is active. A skill built for one tool needs to be manually copied and maintained for the others.

The divergence is not immediately obvious — because each assistant believes it is following the current rules. None of them are following the same rules. Silent divergence is structurally worse than one badly-configured assistant, because at least a single bad configuration is visible.

This article describes what changes when you treat assistant configuration as shared infrastructure rather than per-tool setup — and how three enforcement layers keep everything in sync without a synchronisation step.

**Credibility signal:** The setup described here runs three assistants concurrently on a live codebase: Claude as primary architect, Qwen3.5:9b as local synthesis LLM and pipeline worker, and Trae as IDE assistant. Recovery from a full machine wipe requires one command: `git clone`.

---

## Key takeaways

- A tool-specific root instruction file that only redirects to shared context makes multi-assistant parity achievable without per-tool maintenance.
- Symlinks from each assistant's native directory to one shared harness give all three tools identical skills, hooks, and rules from a single git source.
- Three awareness layers — session context, task-type routing, and file-type rules — create a redundant activation system: when one signal is missed, another catches it.

---

## Who this is for

Developers running more than one AI coding assistant on the same project, or planning to. Also: developers who have built a well-tuned single-assistant setup and want to understand what changes when you add a second tool, a new team member, or need to rebuild from scratch.

---

## What you'll learn

- Why per-tool configuration breaks silently and how to detect the divergence before it causes damage
- The symlink pattern that gives all assistants identical behaviour from a single git-tracked location
- How three enforcement layers work together so that missing one signal does not mean the constraint goes unenforced

---

## The naive approach and where it breaks

The best-published guides for a single assistant describe a well-considered setup: a lean root instruction file, path-scoped rules loaded by file glob, custom skills, hooks, and a curated tool list. That architecture is sound — for one operator and one assistant.

The problem is the assumption it encodes. All of it lives in a tool-specific folder. Configuration written for one assistant is tool-specific by definition. Add a second tool. Add a third. Each accumulates its own directory. Each grows its own rules. None of them share anything.

In practice this produces three failure modes:

**Skill duplication.** A skill built for one assistant does not exist for the others. When those assistants attempt the same task, they improvise.

**Rule fragmentation.** A constraint written for one tool does not run when another tool makes the same edit. The constraint protects one session. The failure remains possible in all others.

**Silent divergence.** Over weeks, each config evolves separately. No assistant knows the others exist. No test detects that they have drifted. This is the worst failure mode: each session gives slightly different results depending on which tool you open, and the cause is invisible.

---

## The inversion: a tool-specific root file as a redirect

The simplest change has the most structural consequence.

The root instruction file for one assistant in this project is:

```markdown
See the repository’s shared AI context for project conventions, assistant roles, skills, memory, and workflow.
```

Two lines. One is a pointer. One is blank.

The point is not minimalism. A tool-specific root file is invisible to every other tool. By making it a pointer, all substantive configuration is forced into a shared, git-tracked directory — tool-agnostic, portable, and readable by every assistant from a single clone.

The root file never drifts from the actual rules, because it contains none. The rules live in one place. The pointer is permanent.

---

## The harness: one location, many consumers

A shared harness directory is the single canonical location for assistant tooling:

```
<shared-harness>/
├── skills/    ← procedural skills
├── hooks/     ← deterministic guardrails
└── rules/     ← path-scoped rule files
```

Each assistant's native configuration directory contains symlinks pointing there:

```
<assistant-A-config>/
├── tool_settings.json  ← tool-specific (permissions, hook syntax)
├── skills  ─────────→  <shared-harness>/skills
├── hooks   ─────────→  <shared-harness>/hooks
└── rules   ─────────→  <shared-harness>/rules

<assistant-B-config>/
├── tool_settings.json  ← tool-specific
├── skills  ─────────→  <shared-harness>/skills
├── hooks   ─────────→  <shared-harness>/hooks
└── rules   ─────────→  <shared-harness>/rules
```

The tool settings file is the only thing that varies per assistant. It holds tool-specific configuration: permission allowlists, hook registration syntax, and other settings. Everything that governs actual behaviour — the skills, hooks, and rules — lives once and is shared.

Adding a new skill requires one change. All assistants see it immediately on their next session start. No synchronisation step. No forgetting to update two of the three.

The symlinks are committed to the repository. Cloning the repo gives you the full harness — no bootstrap script, no manual setup.

---

## Hooks: deterministic guardrails

LLMs produce probabilistic outputs. Hooks add deterministic guardrails that execute reliably regardless of what the assistant decided.

### PostToolUse — Auto-formatter

```json
{
  "matcher": "Write|Edit|MultiEdit",
  "hooks": [{
    "type": "command",
    "command": "if [[ \"$CLAUDE_TOOL_FILE_PATH\" == *.py ]]; then uv run ruff format \"$CLAUDE_TOOL_FILE_PATH\" >/dev/null 2>&1 || true; fi"
  }]
}
```

After every write or edit to a Python file, an auto-formatter runs silently. The file is clean before the next turn. The assistant never accumulates formatting debt or gets confused by its own inconsistent indentation.

The `|| true` is deliberate: if ruff is unavailable, the session continues. The hook degrades gracefully rather than blocking work.

### PreToolUse — Git push gate

```bash
#!/usr/bin/env bash
set -euo pipefail

payload="$(cat)"
cmd="$(printf '%s' "$payload" | jq -r '.tool_input.command // empty')"

case "$cmd" in
  *"git push"*"origin main"*|*"git push"*" main"*)
    jq -nc '{"permissionDecision": "defer", "reason": "Push to main requires human approval."}'
    ;;
  *)
    jq -nc '{"permissionDecision": "allow"}'
    ;;
esac
```

Any command matching a push to `main` returns `"permissionDecision": "defer"`. The session pauses for human review. All other commands pass through immediately.

The gate is a shell script rather than an inline command — auditable, editable, version-controlled. Adding a new gate (deferring force-pushes, production deploys) is a single-line change in one file that immediately applies to every assistant session.

Both hooks live in the shared harness and are shared across all assistants via the symlink pattern.

---

## Three awareness layers

The harness enforces the same project standards through three overlapping layers. The design is deliberately redundant: when one signal is missed, another catches it.

**Layer 1 — Session context.** At session start, the assistant reads the assistant handbook. It knows what the project is, what each folder contains, and where to look for task-specific guidance. This is the "what exists" layer — passive awareness established once per session.

**Layer 2 — Task-type routing.** The shared assistant handbook contains a "Load when" router: a table mapping task types to the minimum set of documents to load. For example: "If you are debugging a retrieval failure, load the retrieval design notes and the lessons log first." This is the "what to load" layer — it prevents loading irrelevant context that happens to share a directory with the edited file.

A task-type router differs from a file-type rule in an important way: it fires based on *intent*, not file path. An assistant working on LLM config tuning should not load the publication workflow rules just because the config YAML lives near a publishing reference. The router makes the loading decision explicit.

**Layer 3 — File-type rules.** Path-scoped rule files in the shared harness fire automatically when the assistant edits a file matching a glob pattern — without requiring the assistant to remember. This is tool-side enforcement, not assistant-compliance.

Five rule categories cover the project:

| Rule | When it loads | What it enforces |
|------|---------------|-----------------|
| Operations | Code and config files | LLMOps baseline: bounded execution, structured observability, degrade modes |
| Retrieval | Retrieval/query/ingest work | Chunking contract, hybrid search invariants, citation ID requirement |
| Governance | Implementation and config | Naming conventions, reuse gate, prohibited terms |
| Publication | Publishing packs and PRDs | Evidence-layer requirement, publication purity |
| Context sync | Shared context files | Keep helper docs in sync when context changes |

The rule files do not duplicate the canonical standards — they reference them. A rule points at the canonical standard and states the invariants that apply to matched file types. The canonical source remains the single authority.

The three layers point at the same standards. None duplicates the others. Layer 2 handles intent-based loading. Layer 3 handles file-type enforcement regardless of intent. Layer 1 ensures cold-starting assistants are oriented before either of the other two fires.

---

## CI enforcement: drift is a test failure

Convention-only rules erode under task pressure. The only class of control that reliably prevents drift is a machine-enforced gate.

A contract test fails CI if:
- A skill is added without referencing a canonical standard
- A skill duplicates standards instead of referencing them (rules must point to the authority, not restate it)
- The reference docs fail to describe the shared configuration baseline

Another drift test fails if a “status table” claims a gate is missing when the gate already exists in the test suite — preventing stale “not yet implemented” statements from surviving past the point they became false.

The governing principle: any rule that matters enough to state must be backed by a test that fails if it drifts. Convention-only rules drift. Tested invariants do not.

---

## Where this diverges from standard single-assistant guides

The best-published single-assistant guides describe a sound setup. A few specific divergences are worth naming because they are non-obvious from first principles.

**Git is the durable memory; local memory is explicitly ephemeral.** Single-assistant guides use the tool's local project memory as a primary store. This project inverts the architecture: a shared context folder in git is the durable memory; the local memory layer holds only in-flight state — active blockers, open run IDs, architectural decisions still being evaluated. Once a decision settles, it moves to the shared context and the local entry is deleted. The boundary is documented; the consequence is tested recovery with near-zero context loss.

**Context management is architecture, not advice.** The published guides offer the tip "keep your root instruction file under 200 lines." That is good advice. This project encodes the same principle structurally: every shared context file has an explicit "Load when" condition. A file without a load condition is context bloat. The rule is not "be disciplined" — it is "the system is shaped so that loading the wrong thing requires deliberate effort."

**Plan Mode was explicitly rejected.** The published guides recommend Plan Mode as a default posture for multi-file changes. This project reviewed that recommendation and rejected it — with documented reasoning. When the context corpus is complete and well-structured, an assistant that reads the task-appropriate files before acting does not need a separate planning phase. Adding a planning tier on top creates ceremony without adding safety — the safety comes from the contracts, not from the planning step.

---

## Tuned vs. governed

| Dimension | Single-assistant tuning | Shared harness |
|-----------|------------------------|----------------|
| **Memory model** | Tool-specific local memory | Shared context in git — portable, recoverable, tool-agnostic |
| **Multi-assistant** | Tool-specific configuration | All assistants share identical harness via symlinks |
| **Drift prevention** | Discipline and consistent prompting | CI gates that fail when contracts drift |
| **Session model** | Long sessions preferred | Short sessions by design — repo holds state, not conversation |
| **Hooks** | Tool-specific | Shared across all assistants from one source |
| **Root config file** | Lean but substantive | One-line redirect — all knowledge lives in shared context |

Tuning improves a single assistant's reliability for a specific developer in a specific session. Governance makes the delivery model portable, multi-assistant, and self-enforcing — independently of any particular tool install or conversation history.

The distinction is not about which approach is "better" in isolation. It is about what the architecture needs to survive: a tool switch, a new team member, a machine wipe.

---

## Recovery: the machine wipe test

The final test for any AI configuration architecture is what survives a machine wipe.

With a tuned single-assistant setup that relies on tool-local memory: you lose weeks of accumulated rules and corrections. Some of it may be reconstructable from the root config file; most of it is gone.

With this setup:

```bash
git clone <repo>
# The harness configuration is committed — no setup script needed
```

One command. Every skill, hook, rule, and operating constraint is there. No bootstrap script. No manual configuration step. No rebuilding from memory.

The harness is a git artefact, not a session artefact. It survives anything the session does not.

---

## Limits / when not to use

**Single-assistant setups.** If you use one AI coding assistant and have no plans to add a second, the overhead of this architecture is not justified. A well-maintained root instruction file is sufficient.

**Short-lived projects.** The harness pays off over months, not days. The investment in a shared context directory, a contract test, and the symlink pattern is worthwhile for projects with ongoing development. For a two-week prototype, it is not.

**When assistants do not share git access.** The symlink pattern requires all assistants to operate in the same git clone. If assistants are running against different clones or branches, the pattern does not apply directly.

**Single-concern codebases.** The task-type router in the assistant handbook is most valuable when the codebase has multiple concerns — retrieval, visualisation, publication, operations. If the codebase is a single module, the router has one entry and the value is minimal.

---

## Repro notes

Run conditions: multiple assistants operate locally on macOS against the same git clone. Hooks register in each tool’s settings file with tool-specific syntax but execute the same shared shell scripts from the harness.

The symlink is not a clever trick. It is the right data model for shared infrastructure: authored once, consumed by all.

---

*Tags: Software Engineering, Artificial Intelligence, Developer Tools, Software Architecture, Claude*
