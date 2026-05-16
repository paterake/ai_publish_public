Governing AI coding assistants is not “prompt harder”

I learned this the day I switched tools.

I’d spent time tightening “non‑negotiable” instructions for one assistant.
Then I tried another assistant on the same repo, asked for a small change, and got a clean diff that still violated the exact rule I thought was “in place”.

Nothing was wrong with the model. My control surface was.

When you work across sessions and assistants, you don’t get one drift. You get multiple, mutually inconsistent versions of “the rules” — each assistant sincerely believing it’s behaving correctly.

The lesson that changed everything: assistants stay correct by passing gates, not by remembering.

This is the inversion:
govern the repo, not the session.

If the constraint matters, make it executable. Make the repo fail closed. Then the assistant can change, the session can reset, and the rules still hold.

And when review is evidence-led (tests, run artefacts, failure signals), you stop debating intent and start verifying outcomes.

Question: what’s one rule in your workflow that should be enforced by a gate instead of a prompt?

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #Trae
