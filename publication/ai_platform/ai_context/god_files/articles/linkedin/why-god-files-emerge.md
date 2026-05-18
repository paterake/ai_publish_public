Why god files keep emerging (even with clear rules)

You can tell an AI coding assistant “one responsibility per file”.
And still end up with a single file that contains everything.

I’ve watched it happen on “small” changes: add one helper, then one more, then “it’s already open” becomes the default.

This isn’t a mystery. It’s a mechanism:

- Nearest-open-file bias: the file already in context becomes the default place to add “one more thing”.
- Task pressure: as the session fills with implementation detail, structural constraints lose weight.
- No failure signal: prose rules don’t fail fast when violated, so accretion is invisible until it’s painful.

Nearest-open-file bias is just locality:
what’s already loaded becomes the attractor.

The distinction that changed how I think about it:

📌 Instructions describe intent. Gates enforce it. Only one of them fails when you need it to.

If you want architectural hygiene to survive AI-assisted work, don’t rely on prose rules. Encode the invariant as something that fails.

Question: what structural rule in your codebase matters enough that it should be a gate, not a guideline?

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #Trae
