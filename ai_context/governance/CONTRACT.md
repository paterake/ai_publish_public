---
name: "ai_publish_public Contract"
---

# Contract — How ai_publish_public Must Behave

This document is a firm contract for how the AI coding assistant maintains and uses `ai_publish_public`.
It exists to prevent drift, duplication, and “which rule wins?” ambiguity.

## 1) Single source of truth

All normative rules for this repo live under `ai_context/`:
- `ai_context/process/ASSISTANTS.md`
- `ai_context/process/TOOL_WORKFLOW.md`
- `ai_context/ai_harness/rules/publication.md`

Content state lives under:
- `publication/<project>/...`

## 2) No repo-local sync tooling

This repo does not contain sync scripts or sync skills. The export is executed from the private `ai_publish` repo and
committed/pushed into this repo.

## 3) Mirror-only repository

This repo must not be used for authoring.

- Do not draft or edit publication markdown here.
- Do not add PRDs, TODO trackers, strategy docs, or internal workflow state.
- Content is exported from `../ai_publish/publication/` and may be overwritten on re-export.

## 4) Allowed content boundary

Only these should appear under `publication/`:
- `**/articles/**/*.md`
- `**/assets/**` (image assets required for publication; no `*.md`)

Do not copy any other directories or files.

## 5) Safety gates

- Do not add absolute paths, private repo paths, or internal tooling paths anywhere in markdown.
- Do not include internal programme identifiers in publishable markdown.
- Keep assistant/process documentation in `ai_context/` only.

## 6) Reporting discipline

When performing an export, report:
- the scope requested (project/area/pack)
- the targets copied
- any missing/empty sources (not an error; just note)
- any copy errors

## 7) Standards propagation (hard)

If any canonical standards in `ai_context/` are changed, update this repo’s process docs in the same pass.
