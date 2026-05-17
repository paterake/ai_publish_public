Three AI coding assistants, zero configuration drift.

I only noticed the problem when I added a second tool.

One assistant was behaving perfectly.
It “knew the rules”.

Then I asked my IDE assistant to make the same kind of change and got a clean diff that still violated the exact constraint I thought was “in place”.

Nothing was wrong with the model.
My rules were living in tool-specific folders the other assistant could not see.

That’s configuration drift:
multiple truths about “the rules”, depending on which tool is active.

The consequence is brutal: you don’t have “one setup”. You have multiple drifting ones — each assistant sincerely believing it’s compliant — and you only find out after a failure.

The fix:
put the rules in the repo once, and make every assistant read from the same file.

In practice:
each assistant’s tool-specific “entry” file becomes a short redirect to shared, tool-agnostic repo context, and skills/hooks/rules live once and are shared across tools via symlinks.

That’s how three tools behave as one.

Question: what’s the first “rule” in your setup that would fail immediately if it drifted — instead of being silently ignored?

#AI #AIEngineering #SoftwareArchitecture #ClaudeCode #Trae
