The assistant applied the right tags — for the wrong project.

Not a bug in the tooling.

A structural decision I hadn't made yet.

When you run an AI-assisted workflow across two projects, the first thing that breaks is context scope. A shared skill file starts collecting project names. A shared tag list grows a new column. The framework quietly becomes a state file with no version history and no owner. Publishing is the worked example here.

Then you open a session for Project B, and the assistant already "knows" things about Project A.

The fix isn't to be more careful.

It's to separate the two things that got merged:

→ Framework (generic, stateless, reusable across every project)

→ State (scoped, per-project, stored separately from the shared framework)

Once that line exists, context load becomes intentional. You open a project session with its state. The framework doesn't carry history from other projects. What the assistant knows at session start is exactly what you chose to give it.

Lesson: if a skill file starts accumulating project names, it's no longer a framework — it's an unversioned state file with no owner.

#ClaudeCode #AICodingAssistant #AgentHarness #SoftwareEngineering #DeveloperTools
