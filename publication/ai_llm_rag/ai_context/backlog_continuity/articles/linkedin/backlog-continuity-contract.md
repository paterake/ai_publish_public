A simple pattern that makes AI work resumable across sessions

Chat history is not a system of record.
That’s why your second AI session can undo your first.

I hit this on a multi-session refactor: the second session was “helpful” in a way the first session would never have been, because the constraints were trapped in the chat.

The consequence is predictable:
you lose decisions, redo work, and waste time reloading context.

This is a state problem:
decisions are durable state, but chat is working memory.

The fix is a backlog continuity contract:
one anchor document per thread, stable anchors per item, and a completion protocol where the assistant updates the durable state after each task (done / constraints / next / verification).

That last part matters most: every completed item establishes constraints that future items must honour. If you don’t forward them explicitly, drift is inevitable.

With the contract in place, resuming becomes a one-liner:

> From `<anchor_doc>`: continue `<item anchor>`.

The key detail: in a harnessed workflow, the continuity loop is automated.
No human in the loop keeping notes between sessions. The harness requires the anchor doc update as part of “done”.

You stop asking assistants to remember. You ask them to read.

Question: do your backlogs currently store durable state, or do they assume the chat will?

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #Trae
