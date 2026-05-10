# Why Two AI Agents Didn't Collide

Two AI coding assistants. Same codebase. Working in parallel.

No merge conflicts. No collision. No coordination protocol.

I didn't plan it that way — it emerged from the structure.

## Key takeaways

- Two AI agents can operate safely in parallel on the same codebase when each file owns exactly one concern and no two agents have reason to touch the same file.
- Artifact boundaries are multi-agent coordination infrastructure — not documentation hygiene. The structure does the coordination; tooling doesn't need to.
- The same architectural principle that prevents merge conflicts in human teams applies to AI agents: shared mutable state is where collisions happen.

## Who this is for

This is for engineers beginning to run multiple AI coding assistants or agent tools in parallel — and for anyone who has wondered whether parallel agent operation requires explicit coordination infrastructure to be safe.

## What happened

I was refactoring a part of my AI publishing harness: replacing a Python script that auto-generated cover image prompts with an intent-contract system.

The new design separated concerns across three artifacts:

- A project-level theme file (`COVER_THEME.md`) holding the fixed visual identity and style language — the things that don't change between articles
- A per-pack PRD section (`Cover Intent`) holding the variable concept — the single creative decision each article requires
- A skill that reads both files and assembles them at generation time

Two AI coding assistants were involved: Claude Code and Trae. Both were active. Both were making changes.

No conflicts emerged.

## Why no collision

The answer isn't coordination tooling. It's structure.

Each file in the codebase owned a single concern. The theme file owned the fixed style. The PRD section owned the concept. The skill owned the assembly logic. Nothing was shared mutable state — there was no single file that both agents needed to write.

Collision happens when two agents (or two developers) need to modify the same file at the same time. When files are structured so that each concern lives in exactly one place, this condition rarely arises. There is no ambiguity about what belongs where, so there is no competition.

The boundary between files did the coordination. Not a lock, not a protocol, not a workflow step. Just the fact that the files were structured correctly.

## What "owning one concern" means in practice

A file owns one concern when you can describe its purpose in a single sentence without using "and."

`COVER_THEME.md` holds the fixed visual identity for this publication's cover images.

`Cover Intent` in each PRD holds the variable concept for this article's cover image.

The skill `create-medium-image-brief` reads both and assembles the prompt.

Each of these is one thing. There is no file that holds both the fixed style and the variable concept. There is no file that holds both the concept and the assembly logic. The concerns are cleanly separated, so the files are cleanly separated, so two agents working simultaneously have no reason to reach for the same file.

This is the principle: separate concerns, separate files, no shared mutable state. It is the same principle that prevents merge conflicts in human teams. It applies just as directly to AI agents.

## The meta-observation

The harness I was refactoring is the harness I use to write articles about AI-assisted development. The articles in this series make the case that artifacts — structured knowledge files — are the primary deliverables in AI-assisted work, and that protecting their structure is the governance obligation, not documentation hygiene.

The refactor I was doing when the parallel operation happened was itself an artifact-boundary refactor: taking a monolithic Python script and replacing it with cleanly separated files, each owning one concern.

Two agents were able to work in parallel without collision precisely because the artifact structure was correct.

The system was demonstrating its own thesis.

## What to design for

If you are beginning to run multiple AI agents or tools in parallel, the question to ask is not "what coordination protocol do I need?" It is "what do my files own?"

A codebase where each file owns exactly one concern is a codebase where parallel agents rarely collide. The structure does the coordination. The files are the infrastructure.

Design the artifacts first. The parallel operation becomes safe as a consequence.
