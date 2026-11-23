Serve the intent, not the method

I asked Claude Code why my rule from May, to prefer maintained libraries over custom code, still had not held. It said: "It keeps slipping because almost everything in how the work runs rewards writing code, and nothing makes adoption the default question."

Coding agents reward themselves for writing code. Given a file lock or an OAuth client to build, the agent writes one, adds tests and passes it itself, even when a maintained library already does the job. I had written the preference for libraries into my agents' rules in May, and it did not hold.

What finally delivered was a change in who signs off on the agent's work. The rules now tell the agent to adopt the recognised tool first and write custom code only for a gap the work cannot do without. A second agent that did not write the code judges each change against what was asked, and a swap to a library is checked against a before-and-after baseline.

Custom code is still rewarded where the product needs it, once it shows the need.

This is one combination on one repository, and the Medium article holds the cases.

When did an agent last pass its own code while a library already did the same job?

#AI #AIEngineering #SoftwareEngineering #ClaudeCode #AIAgents
