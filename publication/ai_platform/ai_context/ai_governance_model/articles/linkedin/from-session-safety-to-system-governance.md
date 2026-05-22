Plan Mode is useful.

It’s also the wrong thing to rely on when the work spans weeks, tools, and sessions.

“Plan first” is a principle.
Plan Mode is just one surface where planning can happen.

The failure modes that break long-running AI-assisted work aren’t “it started coding too early”.
They’re repo-level:

- cross-session drift (old decisions come back)
- cross-assistant inconsistency (different tools, different defaults)
- compaction dilution (constraints summarised away)
- token burn from context bloat (loading “everything” lowers precision)

So the posture has to change:
govern the repo, not the session.

The governance model I ended up with has four layers:

- contracts (module-level): intent, scope, invariants, observable “done”
- constitution (repo-level): cross-cutting non-negotiables and boundary rules
- procedures (workflow): repeatable checklists so execution doesn’t depend on memory
- gates (automation): failure signals that make rules real

The practical test is simple:
close every chat, switch assistants, and continue the work.
If it stays consistent from repo-backed contracts alone, you have governance.
If it doesn’t, you had session memory.

Question: what’s one “rule” in your workflow that should be a gate instead of a prompt?

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #Trae
