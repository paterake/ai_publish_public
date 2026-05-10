# Framework vs. State: The Separation That Keeps Multi‑Project AI Harnesses Correct

We had two active publishing projects. The assistant knew the right tags for both — until the session where it didn't.

Not a hallucination. Not a tooling failure.

A structural decision we hadn't made explicitly: the framework and the project state had quietly merged.

## Key takeaways

- In any multi-project AI harness workflow, context bleed happens when framework rules and project-specific state share the same files.
- The fix is an explicit structural boundary: generic framework in one place, per-project state in another.
- Enforcement scripts that check this boundary make the separation durable, not just aspirational.

## Who this is for

This is for engineers and builders running AI-assisted workflows across more than one project — and who have started to notice that a shared skill or config file is accumulating project-specific knowledge that doesn't belong there. Publishing is the worked example here; the pattern applies to any multi-project harness setup.

## The failure mode: state drifts into the framework

When a harness workflow grows from one project to two, something subtle starts to happen.

A shared skill file gets a tag line added: "for Project A, use this tag set." A shared config picks up a project-specific location. A generic procedure grows a column for Project A's metadata.

Each addition is reasonable in isolation. Collectively, they turn a generic, reusable framework into a stateful file with project-specific dependencies — and no versioning, no ownership, and no clear signal that anything has changed.

The failure surfaces in a session:

You open a session for Project B. The assistant already "knows" Project A's vocabulary. It suggests Project A's tags. It applies Project A's sequencing assumptions. It is coherent. It is wrong.

You fix it manually. The session continues. The root cause stays.

## The decision: an explicit separation

The structural decision that prevents it is not subtle:

- **Framework layer** — generic rules, stateless skills, shared standards. No project identifiers. No per-project defaults.
- **Project state layer** — project-specific metadata, sequencing, tag defaults, post-output logs. Scoped entirely to one project.

Once this line exists, context load becomes intentional. Opening a session for Project B means loading Project B’s state. The framework stays generic. Nothing from Project A bleeds in.

## The mechanism: how the separation is maintained

Three artefacts enforce the boundary in practice.

**A project registry** defines which projects exist in the workflow and where each project’s state lives. It is the authoritative list — adding a project here is what makes it exist in the workflow. It does not contain project-specific rules; it only contains pointers.

**Per-project metadata** holds what would otherwise drift into the framework: tag defaults, output strategy, hashtag policy.

**A validator** enforces the separation mechanically. It checks that project state is not stored alongside shared framework rules, that shared framework rules do not contain project-specific identifiers, and that required per-project state exists. It fails fast on violation.

Without a validator, the separation is a convention. With it, drift becomes a CI failure.

## What the separation buys

**Scoped context load.** Each session loads the framework once and the project state for the relevant project. There is no need to "unload" knowledge from other projects, because it was never loaded.

**No cross-project bleed.** Tags, sequencing rules, and vocabulary are per-project. The assistant cannot apply Project A's defaults to Project B because those defaults are not in the shared context.

**Safe multi-project growth.** Adding a third project means adding new per-project state and registering it. The framework does not change. Existing project state does not change.

## Limits (and what the separation does not solve)

The framework/state separation prevents cross-project bleed. It does not prevent:

- **Duplicated state across projects.** If two projects have the same tag defaults, they will be stated twice. That is fine — duplication across independently scoped state files is not drift.
- **Stale state within a project.** If per-project metadata becomes outdated, the validator will not catch it. That is a review problem, not a structure problem.
- **Framework drift over time.** Shared skills and standards still need maintenance. The separation only prevents project state from polluting the framework — it does not make the framework self-maintaining.

## The lesson

When a framework starts accumulating project names, it is no longer a framework.

It is an unversioned state file with no owner, shared across every project, and wrong for at least one of them at any given time.

The fix is not careful editing. The fix is a structural boundary between what is generic and what is scoped — and a script that enforces it. The publishing workflow is the example; the pattern holds for any multi-project AI harness.
