# Tool Workflow — Sync Publication Content

This workflow applies when updating this repo from `../ai_publish/publication/`.

## Inputs (from the user)

Supported commands:
- `sync articles for <project>`
- `sync articles for <project> implementation`
- `sync articles for <project> ai_context` (also accept `ai-context`)
- `sync articles for <project> implementation <pack>`
- `sync articles for <project> ai_context <pack>` (also accept `ai-context`)
- `pull next publishable for <project> <area>`
- `resync next publishable for <project> <area>`

## Scope rules

- `<project>` is required.
- Packs live under:
  - `publication/<project>/implementation/<pack>/`
  - `publication/<project>/ai_context/<pack>/`
- If an area is provided, limit selection to that area. Otherwise, include both.
- If a pack name is provided, operate on that one pack.

## What to copy

Copy only:
- `articles/` (recursive)
- `assets/` (recursive, if present at source)

Do not copy:
- `docs/`
- `PRD.md`
- `TODO_PUBLICATION.md`
- any other files outside `articles/` and `assets/`

## Procedure

1) Determine the scope from the user command.

2) Enumerate targets from `../ai_publish/publication/<project>/`:
- Full project: all packs under `implementation/` and `ai_context/`.
- Area only: all packs under that area.
- One pack: just that pack under the selected area.

3) For each target, ensure destination directories exist:

```bash
mkdir -p publication/<project>/<area>/<pack>/articles
```

4) Copy `articles/` with rsync:

```bash
rsync -av \
  ../ai_publish/publication/<project>/<area>/<pack>/articles/ \
  publication/<project>/<area>/<pack>/articles/
```

5) If `assets/` exists for the pack, copy it too:

```bash
if [ -d "../ai_publish/publication/<project>/<area>/<pack>/assets" ]; then
  rsync -av \
    ../ai_publish/publication/<project>/<area>/<pack>/assets/ \
    publication/<project>/<area>/<pack>/assets/
fi
```

6) Report (see `ai_context/governance/CONTRACT.md`).

## Constraints

Enforce the contract:
- `ai_context/governance/CONTRACT.md`

## Next publishable (READY pack only)

When the user asks for “next publishable”, sync only one pack selected from:
- `../ai_publish/publication/<project>/implementation/<pack>/`
- `../ai_publish/publication/<project>/ai_context/<pack>/`

Selection:
1) Read `../ai_publish/publication/<project>/docs/PUBLICATION_PATH.md`.
2) Choose the first entry with `Status: READY` matching the requested `<area>`:
   - `implementation` → pack starts with `implementation/`
   - `ai_context` or `ai-context` → pack starts with `ai_context/`
   - `any` → either prefix

Copy rules (for the chosen pack only):
- Copy `articles/linkedin/**/*.md`
- Copy `articles/medium/**/*.md`
- Copy image assets under `assets/` (exclude `*.md`, including cover briefs)

Do not copy:
- `PRD.md`, `TODO_PUBLICATION.md`, `docs/`, or any other files outside the paths above
