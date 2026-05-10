I run three AI coding assistants on the same project.

When I added the second tool, the same task started producing different results depending on which was active — not dramatically, just subtly wrong. Code that compiled and violated a constraint I thought was in place.

The constraint was documented. The problem was structural: the rule lived in a directory the second tool couldn't see.

That's a harness problem. Not a model problem.

Claude Code, Cursor, Trae — these aren't interfaces for talking to AI. They're operating systems for AI work. The model is the CPU. The harness is the OS: context loading, tool access, execution hooks, permission scoping, memory, session lifecycle. Without the OS, the CPU is capable and unusable.

The non-obvious part:

The model sets the ceiling on output quality. The harness sets the floor.

A governed harness with a moderate model consistently outperforms an ungoverned harness with a stronger model on structured work.

📌 Most engineers frustrated with inconsistent AI coding results are trying to fix the model. They should be fixing the operating system.

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #Trae

Question: when your AI coding tool produces unexpected output — what's your first instinct? Prompt, tool, or setup?
