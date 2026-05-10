# AI Assistant Guide — ai_publish_public

**Context for all coding assistants working in this repository.**

**Read this before starting any work.**

## What this repository is

`ai_publish_public` is a **public mirror of finished publication drafts** from the private `ai_publish` repository.

Its role is simple:
- copy publish-ready markdown (and required assets) into a public surface
- so the human can import into LinkedIn and Medium without exposing the private repo

This repo is intentionally thin. It should not accumulate process state.

## Source of truth

All publication content originates in:
- `../ai_publish/publication/`

Do not edit publication content directly in this repo. Any direct edits will be overwritten on the next export from
`ai_publish`.

## What belongs here (and where)

This repo should contain only:
- `publication/<project>/**/articles/**/*.md`
- `publication/<project>/**/assets/**` (only assets required for publication, e.g. cover images)

Typical layout:

```
publication/<project>/implementation/<pack>/articles/
  medium/     ← long-form Medium drafts
  linkedin/   ← companion LinkedIn posts

publication/<project>/implementation/<pack>/assets/
  *.png

publication/<project>/ai_context/<pack>/articles/
  medium/
  linkedin/

publication/<project>/ai_context/<pack>/assets/
  *.png
```

Everything else stays private in `../ai_publish/` (PRDs, TODOs, strategy, internal workflow state).

## Canonical docs in this repo

This repo’s rules live in:
- `ai_context/governance/CONTRACT.md`
- `ai_context/process/TOOL_WORKFLOW.md`
- `ai_context/ai_harness/rules/publication.md`

This repo does not run sync tooling. Exports are executed from the private `ai_publish` repo and committed/pushed into
this repo.

## Working model

1. Draft/review in `../ai_publish/` (private)
2. Export into this repo (public mirror)
3. Import from this repo into LinkedIn / Medium

During sync:
- copy faithfully (no editorial changes)
- copy only publishable artefacts (articles + required assets)

## Standards propagation (hard)

If canonical standards in `ai_context/` change, update any dependent skills/pointers in the same pass.
