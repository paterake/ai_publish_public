# Tool Workflow — Sync Publication Content

This workflow applies when updating this repo from `../ai_publish/publication/`.

Exports are executed from the private `ai_publish` repo and committed/pushed into this repo.

## Inputs (from the user)

Supported requests:
- “export the next publishable pack”
- “export `<project>` `<area>` `<pack>`”

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
- image assets under `assets/` (recursive, if present at source; exclude `*.md`)

Do not copy:
- `docs/`
- `PRD.md`
- `TODO_PUBLICATION.md`
- any other files outside `articles/` and `assets/`

## Procedure

1) Determine the scope from the user command.

2) Run the deterministic export script from the `ai_publish` repo root:

```bash
uv run python scripts/sync_public_mirror.py next --project <project> --area any
```

3) Report (see `ai_context/governance/CONTRACT.md`).

## Constraints

Enforce the contract:
- `ai_context/governance/CONTRACT.md`

## Next publishable (READY pack only)

When the user asks for “next publishable”, export only one pack selected from the requested `<area>` based on
`publication/<project>/docs/PUBLICATION_PATH.md` in the private `ai_publish` repo.
