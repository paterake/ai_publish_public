Your AI assistant’s memory isn’t backed up

Most advice about AI coding assistants focuses on one question:
can you trust what the assistant produces?

The failure mode that costs more time is quieter:
your rules don’t transfer (Claude ↔ Trae ↔ Qwen).

I hit this the hard way: I spent weeks “training” one assistant, then switched tools and watched it regress to day one.

Nothing got worse.
I just realised I’d stored the rules in a place only the first tool could see.

The consequence is predictable: switching assistants becomes a reset, and you pay the cost again in every session.

The fix is a simple memory split:
keep in-flight context ephemeral, but store stable operating rules as shared, git-backed context that every assistant can read.

One useful way to sanity-check what you’re storing is “knowledge type”:
working (in-flight), semantic (timeless facts), episodic (what happened), procedural (how-to routines).

📌 The lesson: if a rule should survive a restart, it must live in the repo. Otherwise it isn’t a rule.

Question: where do your “how we work here” rules live today — in the repo, or inside one tool?

#AI #SoftwareEngineering #DeveloperTools #ClaudeCode #Trae
