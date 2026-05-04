# AI Assistant Guide — ai_publish_public

**Context for all coding assistants working in this repository.**

**Read this BEFORE starting any work.**

---

## What this repository is

`ai_publish_public` is a **public mirror of article content** from the private `ai_publish` repository.

It exists for one purpose: to make finished article drafts accessible for importing into LinkedIn
and Medium, without exposing the private `ai_publish` repo (which contains unpublished drafts,
publishing strategy, PRDs, and internal workflow state).

**This repo contains article content only.** No PRDs, publishing strategy, TODO trackers, or
internal metadata. Those all live in `../ai_publish/`.

---

## Source of truth

All article content originates in `../ai_publish/publication/`. Do not edit articles directly in
this repo — edit in `ai_publish`, then re-sync using the `sync-articles` skill.

**Source of truth:** `../ai_publish/publication/`

---

## What is here

```
publication/<project>/<module>/articles/
  medium/     ← long-form Medium drafts
  linkedin/   ← companion LinkedIn posts

publication/<project>/root/<topic>/articles/
  medium/     ← long-form Medium drafts
  linkedin/   ← companion LinkedIn posts

publication/<project>/root/<topic>/assets/
  *.png       ← cover images (synced alongside articles)
```

Current projects:

| Project | Source |
|--------|--------|
| `ai_llm_rag` | `../ai_publish/publication/ai_llm_rag/` |

---

## Skills

| Skill | When to invoke |
|-------|---------------|
| `sync-articles` | Copy article drafts from `../ai_publish` into this repo |

---

## Working model

This repo is **read-only for authoring**. The workflow is:

1. Draft and review articles in `../ai_publish/`
2. Run `sync-articles` to copy finished drafts here
3. Import from this public repo into LinkedIn / Medium

**Never author or edit articles here.** Any changes made directly will be overwritten on the
next sync.

---

## Solo Author Context

One human author. One or more AI assistants.

The author provides editorial direction. AI assistants copy content faithfully from `../ai_publish`
and report exactly what was synced. No editorial judgment or content modification during sync.
