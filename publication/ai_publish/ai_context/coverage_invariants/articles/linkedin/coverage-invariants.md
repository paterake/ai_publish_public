The easiest way to lose a requirement in an AI workflow is to add an artifact and never map it.

Nothing breaks.

The assistant doesn't "forget" in the human sense.

The artifact just never gets loaded again, because nothing in the workflow points to it.

In traditional development, losing a doc is a filing problem. You can re-derive what it said from the code.

In AI-assisted development, the artifact is the requirement. Lose it, and the code it governed keeps running — without the constraint that defined it. The requirement silently stops existing.

That's drift by omission — and it's worse than wrong artifacts because it fails silently.

So I enforce a simple coverage invariant:

→ Every source artifact must be mapped to at least one publishing plan, or explicitly excluded with a one-line rationale.

No third state.

Lesson: if it isn't mapped, it doesn't exist. Make unmapped artifacts fail loudly, or you'll lose requirements without knowing it.

#ClaudeCode #AICodingAssistant #AgentHarness #SoftwareEngineering #DeveloperTools
