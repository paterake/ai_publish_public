My three AI coding assistants (Claude Code, Trae, and Qwen Code) share the same rules, skills, and enforcement hooks.

CLAUDE.md is one line.
TRAE.md is one line.
QWEN.md is one line.

All three point to the same governance layer. Rules and skills are symlinked across all three. A skill written once is available to every coding assistant.

I didn't design this on paper. I hit a rate limit.

Mid-session, Claude Code stopped. I switched to Qwen Code. Every rule I'd spent months encoding was gone. Not in the repository. Never was.

More failures followed:

→ Long sessions eroded constraints I'd clearly stated
→ Validation work from late-night sessions vanished before the next one
→ Switching tools meant re-establishing weeks of context from scratch

Different triggers. One root cause: project state that lived outside the repository.

The backlog fix is equally simple:

> Anchor. Next: Item 3.
> To resume: From docs/TODO.md: continue Item 3.

One command. Any assistant. Cold start. No shared chat history needed.

The result: 12 modules, 6 months, three AI coding assistants cycling in and out. Not one solution locked to any tool's memory.

One of those tools, Trae, was set to auto mode: it switches the underlying model dynamically based on task and speed. Governance held through that too. Whatever model it selected read the rules cold and applied them.

Coding assistants are hot-swappable when the platform is the memory.

Model quality sets the ceiling.
Governance quality sets the floor.

Harness features converge. Governance compounds.

Full piece → [link]
