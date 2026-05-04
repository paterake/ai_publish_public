---
name: "sync-articles"
description: "Copy article markdown files (and cover assets) from ../ai_publish/publication/ into this public repo. Invoke when user says 'sync articles for <module>', 'sync articles for <module> <implementation>', or 'sync articles for <module> root'."
---

# Skill: Sync Articles to Public Repo

## Purpose

Copy article markdown files — and cover image assets — from `../ai_publish/publication/` into
this repo (`ai_publish_public`). Only article content is copied. No PRDs, TODO files, or internal
tracking metadata are ever placed in this public repo.

This keeps `ai_publish_public` up to date so articles can be imported into LinkedIn/Medium from
a public repository.

**Source of truth:** `../ai_publish/publication/` — never edit articles directly here.

---

## When to Invoke

| User command | Scope |
|---|---|
| `sync articles for <module>` | All implementations + all root topics under `publication/<module>/` |
| `sync articles for <module> <implementation>` | One implementation only |
| `sync articles for <module> root` | All root topics under `publication/<module>/root/` |
| `sync articles for <module> root <topic>` | One root topic only |

Example: `sync articles for ai_llm_rag elt_llm_explore`

---

## What to Copy

### Implementation modules

**Source:** `../ai_publish/publication/<module>/<implementation>/articles/`
**Target:** `publication/<module>/<implementation>/articles/`

Copy everything under `articles/` recursively. Typical structure:

```
articles/
  medium/     ← *.md files
  linkedin/   ← *.md files
```

### Root topics

**Source:** `../ai_publish/publication/<module>/root/<topic>/articles/`
**Target:** `publication/<module>/root/<topic>/articles/`

**Source assets:** `../ai_publish/publication/<module>/root/<topic>/assets/`
**Target assets:** `publication/<module>/root/<topic>/assets/`

Copy `articles/` and `assets/` (if the assets directory exists in source).

---

## What NOT to Copy

| Path pattern | Reason |
|---|---|
| `PRD.md` | Internal publishing strategy |
| `TODO_PUBLICATION.md` | Internal workflow state |
| `docs/` | Internal tracking and metadata |
| Any file outside `articles/` or `assets/` | Not article content |

---

## Procedure

### Step 1 — Determine scope

Parse the user's command to identify:
- `<module>` — required (e.g., `ai_llm_rag`)
- `<implementation>` — optional specific implementation (e.g., `elt_llm_explore`)
- `root` — if present, target root topics instead of implementation modules
- `<topic>` — optional specific root topic (e.g., `above_the_loop`)

### Step 2 — Enumerate targets

**Full module sync** (`sync articles for <module>`):
1. List all implementation directories: `../ai_publish/publication/<module>/*/`
   - Exclude `root/` and `docs/` from enumeration
2. List all root topics: `../ai_publish/publication/<module>/root/*/`

**Targeted sync:**
- Single implementation: `../ai_publish/publication/<module>/<implementation>/`
- Root only: all entries under `../ai_publish/publication/<module>/root/`
- Single root topic: `../ai_publish/publication/<module>/root/<topic>/`

### Step 3 — Create destination directories

Before copying, ensure destination directories exist:

```bash
mkdir -p publication/<module>/<implementation>/articles
# or
mkdir -p publication/<module>/root/<topic>/articles
```

### Step 4 — Copy articles

**For each implementation module:**

```bash
rsync -av \
  ../ai_publish/publication/<module>/<implementation>/articles/ \
  publication/<module>/<implementation>/articles/
```

**For each root topic:**

```bash
rsync -av \
  ../ai_publish/publication/<module>/root/<topic>/articles/ \
  publication/<module>/root/<topic>/articles/
```

If `assets/` exists at source, also copy it:

```bash
if [ -d "../ai_publish/publication/<module>/root/<topic>/assets" ]; then
  rsync -av \
    ../ai_publish/publication/<module>/root/<topic>/assets/ \
    publication/<module>/root/<topic>/assets/
fi
```

### Step 5 — Report

After all rsync operations, report:

- Targets synced (list each module/topic)
- Files added or updated (summarise rsync output)
- Source paths that were empty / had no articles yet (no action taken, noted)
- Any errors encountered

---

## Constraints

- Do not copy `PRD.md`, `TODO_PUBLICATION.md`, `docs/`, or any file that is not under
  `articles/` or `assets/`.
- Do not modify article content during copy — faithful reproduction only.
- If a source `articles/` directory is empty, note it in the report but do not fail.
- Run all rsync commands from the root of this repo (`ai_publish_public/`).
