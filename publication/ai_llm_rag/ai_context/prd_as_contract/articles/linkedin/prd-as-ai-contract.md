A PRD written as “documentation about code” can’t govern an AI coding assistant.

Most PRDs assume a human reader will fill in gaps, infer intent, and open the repo when something is ambiguous.

An assistant won’t.
It executes what you described.

That’s why the PRD has to change shape.

📌 An AI-ready PRD is a contract for what must be true, independent of how the implementation achieves it.

Five non-negotiables I now look for:

- contract-first: prevents the assistant from having to infer intent — the contract states inputs, outputs, and invariants explicitly
- code-blind: stops the assistant re-implementing instead of inheriting — no file paths, no function names
- boundary-explicit: stops scope creep before it starts — what this module owns vs delegates is stated, not assumed
- determinism with conditions: prevents silent failure — same inputs + same config must produce the same outputs, and that must be verifiable
- trust posture: bounds non-determinism — caps, retries, degrade modes, and “insufficient evidence” behaviour are defined, not discovered at runtime

The line that prevents the most drift is often a boundary:
non-core modules must not re-define shared platform plumbing. Inherit it and move on.

Which of the five properties is missing from the last PRD you handed to an AI assistant?

#AI #AIEngineering #ProductManagement #ClaudeCode #Trae
