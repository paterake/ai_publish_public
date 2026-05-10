I had a 730-line AI assistant tracker. I replaced it with a 37-line generated table.

The tracker worked at 4 articles. By 27 publication packs, every session loaded 730 lines to answer one question: what should I work on next?

That's a design problem. And the fix is not a better tracker.

AI coding assistants load context fresh every session. A rich narrative document that reads well for humans is often the worst possible format for AI navigating project state. Every line loaded is a cost.

The pattern I now apply:

— Pre-compute expensive work once (user-triggered). The AI reads the output, not the scan.

— Single source of truth, generated projections. Each PRD owns its sequencing. A script extracts a 37-line priority table. If any PRD is missing the field, the script exits non-zero — a self-auditing gap.

— Size context to the task. The AI needs that 37-line table and one PRD. Not the history of every other pack.

— Human commands the expensive step. The AI executes against the result.

Lesson: don't build AI assistants that re-derive what you can pre-compute.

What's the largest context problem you've hit with AI coding assistants?

#ClaudeCode #AICodingAssistant #AgentHarness #DeveloperTools #SoftwareEngineering
