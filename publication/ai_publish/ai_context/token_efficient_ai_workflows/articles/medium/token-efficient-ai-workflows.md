# What a 730-Line Tracker Taught Me About AI Assistant Design

I had a 730-line tracker document. It tracked everything: article status, publishing metadata, proof points, sequencing rationale, resync instructions, dependencies. It was a genuinely useful document when the project had four articles. By the time it had 27 publication packs, every AI coding assistant session started by loading 730 lines to answer one question: what should I work on next?

That is a design problem — and not one specific to publishing. The same failure appears in any multi-step AI-assisted process: data pipelines, build workflows, documentation systems, anywhere an assistant is making repeated decisions from accumulated state.

The fix was not a better tracker. The fix was rethinking what an AI assistant actually needs to load to do its job.

---

## Key takeaways

- Treat context size as a workflow architecture decision, not a prompting detail.
- Pre-compute expensive scans once; make the assistant read compact projections.
- Put state in one place (per-pack PRDs) and generate “what’s next?” tables.

## Who This Is For

Developers building workflows where an AI coding assistant manages a multi-step process across sessions. Publishing pipelines, documentation systems, content generation workflows, data engineering pipelines with AI-managed orchestration. Anywhere your AI assistant loads context, makes a decision, takes an action — and then does it again tomorrow, having forgotten everything from today.

## What You'll Learn

- Why context size is a workflow architecture decision, not a prompt engineering problem
- Four principles for structuring AI-assistant workflows to minimise per-session token burn
- A concrete before/after from a real refactor: 730 lines → 37 lines

---

## The Problem: Loading Cost per Session

AI coding assistants effectively treat each session as a cold start unless you give them durable, portable context to load. Tool memory may exist, but it’s vendor-controlled and non-portable across assistants. Everything the assistant needs to understand the current state of your project must fit into what it reads at the start of the session — or be re-derived from scratch.

This creates a loading cost. Every line of context the assistant reads takes time, costs tokens, and increases the chance of irrelevant context crowding out relevant context. It also introduces a subtler problem: a rich, narrative document that makes perfect sense to a human may be the worst possible format for an AI assistant trying to answer "what is the current status of pack X?"

My tracker had grown into exactly that. Each of the 27 publication packs had its own section: status, file pointers, publishing metadata, the story, proof points, sync instructions, dependencies. Useful for a human reading the file top to bottom. Expensive for an AI loading it to find the one field it needs.

The naive fix was to improve the tracker — add structure, reduce duplication, tighten the format. I tried that. The problem is not the format. The problem is that a central tracker is the wrong architectural pattern for AI-assistant workflows.

---

## The Pattern: Pre-Compute, Project, Load-on-Demand

The insight came from noticing three things about how the AI assistant actually used my tracker:

1. **It read the whole file to answer questions about one pack.** The sequencing, the status, the metadata — all of it was interleaved in a 730-line narrative. To find the status of a single pack (for example, an LLM-based classification module), the assistant read past 300 lines of other content first.

2. **It re-derived the same things every session.** "What's the publish order?" is not a question that changes unless a human changes it. But the assistant answered it by reading every pack's status and inferring the sequence from prose descriptions. Every session.

3. **The expensive work was happening at query time.** Scanning source docs for coverage gaps, inferring sequencing from narrative, cross-referencing article files against tracker entries — all of this happened when the AI read the tracker. It could have happened once, offline, and been stored as a compact result.

These observations led to four principles.

---

## Four Principles

### 1. Separate computation from lookup

Expensive work — scanning a source repo for coverage gaps, checking which files changed since last run, computing which source docs are not referenced by any publication PRD — should happen once, triggered by the human, and written to a durable output.

The AI reads the output. It does not repeat the scan.

In my setup, this means rebuilding a source index snapshot when the source documentation changes. The snapshot pre-computes what changed and what is currently unmapped. The gap analysis reads those fields. It does not rescan the source repo.

The human commands the expensive step. The AI reads the cheap output.

### 2. Single source of truth with generated projections

The 730-line tracker was a copy. Status, metadata, sequencing — all of it duplicated from the individual publication PRDs, where it originally lived. Duplication without CI enforcement always drifts.

The fix: put data in exactly one place and generate projections from it.

Each pack PRD now has a small “publishing order” section: a priority and (optionally) a dependency. A deterministic script parses every PRD, extracts those fields, sorts by priority, and writes a short “what’s next?” table:

```
| Priority | Pack | Status | Publish after |
|----------|------|--------|---------------|
| 15       | guardrails | READY | — |
| 25       | token-efficient workflows | READY | guardrails |
| 35       | evidence layer | READY | token-efficient workflows |
...
```

The PRD is the source of truth. The table is a projection. The AI reads the table to find what to draft next. If something is wrong, it edits the PRD and regenerates.

If a PRD is missing publishing order, the generator fails and lists the gap. The sequencing is now deterministic and self-auditing.

### 3. Context sized to the task

When drafting an article, the AI needs: the publish order table (to find what's next), the specific PRD for that pack (story, source material map, proof points), and the source docs listed in that PRD. That is all.

It does not need the post-publish log for other packs. It does not need the sync history. It does not need the metadata for packs that are not being worked on today.

The old tracker forced the AI to load all of that — because it was all in one file.

The new structure makes the right scoping easy. The 37-line table tells the AI which pack to work on. The individual PRD tells it what to read and what to say. The source docs provide the evidence. Each of these is loaded as needed, not preloaded as a bundle.

### 4. Human commands the expensive step; AI reads the result

This is the operational expression of principle 1. The boundary is explicit:

- **Human triggers:** rebuilding the source index snapshot (expensive; scans documentation)
- **AI reads:** the source index snapshot (cheap; pre-computed fields)

- **Human directs:** "move this pack earlier"
- **AI executes:** edits the pack PRD’s publishing order fields, regenerates the table

- **Human commands:** "draft the next ready article"
- **AI reads:** the generated table (tens of lines) → the specific PRD → the source docs → drafts

The human controls when expensive operations run. The AI is never in a position where it needs to rescan or re-derive to answer a question the human could have pre-computed.

---

## The Result

The session model now has four explicit steps:

1. **Source sync** (human-triggered, on demand): rebuild the source index snapshot
2. **Gap analysis** (AI reads the pre-computed index): the audit outputs a short gap report when clean
3. **Sequencing** (human-directed, AI updates PRDs): the publication-order table is regenerated
4. **Article generation** (human commands, AI executes): reads 37-line table + one PRD + source docs

The 730-line tracker became an ≈60-line file containing only what cannot be derived: the post-publish log and the sync history.

The sequencing table is generated from structured PRD data. Any PRD without publishing order causes the generator to fail — a self-auditing gap detector.

Coverage analysis that previously required reading the big tracker plus scanning sources now runs as a single pre-computed audit over an index snapshot.

---

## The Generalisation

This pattern appears in any multi-session AI-assisted workflow where:

- The assistant answers the same structural questions across sessions ("what is the status of X?", "what should I work on next?", "what has changed since last time?")
- The answers to those questions change only when a human changes them — not spontaneously
- The underlying data that answers those questions requires non-trivial computation to derive

The naive response to a growing context problem is to compress the context: shorter sentences, fewer words, tighter formatting. That is a dead end. The correct response is to stop requiring the AI to derive what you can pre-compute.

**Don't build AI assistants that re-derive what you can pre-compute. Size context to the task, not the project.**

A tracker that is perfect for a human to read is often the worst possible structure for an AI to navigate. Separate the two. The human gets the narrative. The AI gets the projection.

The same principle extended further than I initially expected. After applying it to source coverage and publication sequencing, the same forced-scan problem appeared at the governance layer: the structure of the project's backlog wasn't documented in the assistant's load context, so every new session had to scan the directory tree to discover it. A short addition to the assistant context eliminated that scan — same pattern, different layer.

---

## Limits

This pattern works when the structured data (the PRD priority fields, the JSON pre-computation) stays current. If PRDs are edited but the projection tables are not regenerated, the AI reads stale data. The deterministic scripts help — they exit non-zero on gaps — but they require the human to run them. The discipline of "human commands the expensive step" only holds if the human commands it.

The pattern also assumes the AI is reading from a known, version-controlled structure. It does not help with ad hoc queries against unstructured data.

---

## Key Takeaways

- Context size is a workflow architecture decision. Every line an AI loads per session is a design choice — not a limit to compress around.
- Separate computation from lookup. Expensive scans happen once, triggered by the human. The AI reads the pre-computed output.
- Generate projections from a single source of truth. Duplication without enforcement drifts.
- Size context to the task. The AI needs the 37-line table and one PRD — not the full history of every pack.
- Human commands the expensive step. The AI executes against the result.
