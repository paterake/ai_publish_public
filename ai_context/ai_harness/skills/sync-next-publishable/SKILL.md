---
name: "sync-next-publishable"
description: "Sync the next READY pack (LinkedIn + Medium drafts and image assets only) from ../ai_publish/publication/<project>/ into this public repo."
---

# Skill: sync-next-publishable

## Purpose

Sync only the next publishable pack into this public mirror repo:
- LinkedIn draft(s)
- Medium draft(s)
- image assets only

## Behaviour

- Follow `ai_context/process/TOOL_WORKFLOW.md` (Next publishable section).
- Enforce the constraints in `ai_context/governance/CONTRACT.md`.
- Use `../ai_publish/publication/<project>/docs/PUBLICATION_PATH.md` to select the next pack.

## When to invoke

Supported commands:
- `pull next publishable for <project> <area>`
- `resync next publishable for <project> <area>`

Where `<area>` is one of:
- `implementation`
- `ai_context` (also accept `ai-context`)
- `any` (both areas)
