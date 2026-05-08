---
name: "sync-articles"
description: "Copy article markdown files (and required assets) from ../ai_publish/publication/ into this public repo. Invoke when user says 'sync articles for <project>' or targets a specific area/pack."
---

# Skill: sync-articles

## Purpose

Sync publish-ready content from `../ai_publish/publication/` into this public mirror repo.

## Behaviour

- Follow `ai_context/process/TOOL_WORKFLOW.md`.
- Enforce the constraints in `ai_context/governance/CONTRACT.md`.
- Copy only publishable artefacts (articles and required assets) without modification.

## When to invoke

Supported commands:
- `sync articles for <project>`
- `sync articles for <project> implementation`
- `sync articles for <project> ai_context` (also accept `ai-context`)
- `sync articles for <project> implementation <pack>`
- `sync articles for <project> ai_context <pack>` (also accept `ai-context`)
