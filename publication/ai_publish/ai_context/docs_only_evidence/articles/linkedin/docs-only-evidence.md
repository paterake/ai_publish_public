If you need to read implementation code to produce your output, the output isn't ready yet.

That sounds backwards to engineers, because the code is "the truth".

But in AI-assisted development, this is inverted.

Code is now cheap — an AI generates it from direction. The artifact (the governance rules, implementation definitions, constraints encoded in structured markdown) is the primary deliverable. Code is the by-product derived from it.

Producing output by code-reading pushes you into two quiet failure modes:

→ You leak private structure (paths, names, contracts) because it's now in the writing context.

→ You publish inferred behaviour that was never stated explicitly anywhere, so it can't be audited or maintained later.

So I keep the workflow artifact-first by default. Publishing is the worked example here:

→ Drafts are grounded in source artifacts (README/PRD/governance docs), not implementation scans.

→ Missing proof becomes explicit: a NEEDS_DATA placeholder that names the publish-safe evidence required.

The non-obvious upside is that safety and efficiency align: bounded inputs reduce both leakage risk and rescanning cost.

Lesson: "truth by code-reading" is a governance smell. Produce outputs from explicit artifacts, not inference.

#ClaudeCode #AICodingAssistant #AgentHarness #SoftwareEngineering #DeveloperTools
