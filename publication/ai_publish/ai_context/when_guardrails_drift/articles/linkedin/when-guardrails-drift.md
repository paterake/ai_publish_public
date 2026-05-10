I built a governance harness to stop AI coding assistants from drifting.

Then I found the harness itself drifting.

The same LinkedIn formatting rules were living in four different documents, each slightly different. Every copy looked “helpful”. That’s exactly why it was dangerous: each assistant could follow a different truth and still feel compliant.

So I treated the harness like any other system that suffers drift:

— One canonical home per rule (format, voice, safety).

— Every other file becomes a one-line pointer, not a second copy.

— A mechanical audit fails loudly if duplication creeps back in.

The moment this clicked was seeing the audit end with:

“14 files checked — all clean.”

That’s the real goal: correctness that doesn’t depend on memory, context, or goodwill.

Lesson: if your harness can’t detect its own drift, it isn’t fully engineered.

What’s the quiet drift pattern you’ve seen in your own assistant workflow?

#ClaudeCode #AgentHarness #AICodingAssistant #SoftwareEngineering #DeveloperTools

