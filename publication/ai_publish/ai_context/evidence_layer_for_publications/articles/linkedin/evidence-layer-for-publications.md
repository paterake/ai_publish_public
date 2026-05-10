The easiest hallucination to ship is a real metric with missing evidence.

I used to think my guardrails were enough.

No client names. No private paths. Explicit data-gap markers where proof points were missing.

Then I hit a more mundane failure mode.

Some numbers in my drafts were real at the time. They came from real runs.

But after a big refactor, the run outputs were wiped and reorganised. The prose survived. The artefacts didn’t.

Nothing malicious happened.

The repo just lost the ability to prove its own claims.

That’s what “fake confidence” looks like at an output boundary (publishing is the worked example):

→ a neat table with no run identity
→ a “measured 14×” improvement with no artefact pointer
→ a runtime claim that can’t be replayed to a manifest

The fix wasn’t “be more careful”.

It was to treat claims like code.

I added an evidence layer:

- Any measurement-style claim must be anchored to a `run_id` + artefact pointer (manifest, export, screenshot provenance).
- If it isn’t, it must be explicitly marked as a data gap (NEEDS_DATA) or an evidence gap (EVIDENCE_GAP).
- A deterministic audit scans the markdown and flags any numeric claims without an anchor.

The point isn’t to catch lies.

It’s to make it impossible to ship claims the repo can’t prove.

If you’re building with agent harnesses, this is the missing layer between “the system is governed” and “the write‑up is trustworthy”.

Could your repo prove every published claim from artefacts right now — or are some metrics now orphaned from their runs?

#ClaudeCode #Trae #Qwen #AICodingAssistant #SoftwareEngineering
