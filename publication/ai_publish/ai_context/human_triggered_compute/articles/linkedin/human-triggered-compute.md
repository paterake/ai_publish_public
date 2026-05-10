Asking an assistant "what should I publish next?" is fine.

Watching it re-scan 200 source files to answer — when a 37-line file already had the answer — is a delegation problem.

The issue isn't capability. The assistant can scan those files. It's fast enough. The issue is that it shouldn't need to, and more importantly: it shouldn't decide to.

Source indexing, coverage audits, build steps that rescan large directory trees — these are commands, not queries. They're expensive, their scope is consequential, and they produce outputs that everything else depends on.

The pattern that works:

→ Human triggers the script.

→ Assistant reads the pre-computed result.

Step 1 is a command. Step 2 is a read.

When you separate them, two things happen:

The session gets cheaper (reading a 37-line projection vs. re-deriving it from 200 files).

The scope stays visible (the human decided what got rescanned, not the assistant).

Lesson: human-triggered compute isn't a performance trick. It's a safety boundary — when the human decides what gets rescanned, scope stays visible and controllable.

#ClaudeCode #AICodingAssistant #AgentHarness #SoftwareEngineering #TokenEfficiency
