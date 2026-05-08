---
description: Public mirror rules for publication content and sync tooling.
globs:
  - "publication/**/*.md"
  - "ai_context/**/*.md"
---

# Publishing Rules — ai_publish_public

Canonical standards live in:
- `ai_context/process/ASSISTANTS.md`
- `ai_context/governance/CONTRACT.md`
- `ai_context/process/TOOL_WORKFLOW.md`

## Safety gates

- Do not author or edit publishable drafts in this repo; only sync them from `../ai_publish/publication/`.
- Do not add absolute file paths, private repo paths, or internal tooling paths anywhere in markdown.
- Do not include internal programme identifiers in publishable markdown.
- Copy only `articles/**` and (where present) required `assets/**`. Do not copy PRDs, TODO trackers, strategy docs, or internal docs.

## Workflow gates

- Skills under `ai_context/ai_harness/skills/` stay orchestration-only: reference canonical docs instead of restating rules.
- Keep documentation single-sourced; prefer pointers over duplicated skill text.
- If canonical standards change, update dependent skills/pointers in the same pass.
