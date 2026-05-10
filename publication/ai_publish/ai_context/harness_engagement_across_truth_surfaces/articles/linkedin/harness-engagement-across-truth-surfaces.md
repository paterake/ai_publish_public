The same harness can prevent two different hallucinations: wrong code, and wrong confidence.

If you build with an agent harness, you probably think about hallucinations as a code problem.

Bad diffs. Broken tests. Regressions.

That’s real, but it’s not the only truth surface you’re governing.

When you publish, your readers can’t run your test suite. They can’t inspect your diffs. They are consuming claims.

And AI assistants are very good at removing draft smell.

So you get a quieter failure mode:

→ polished prose
→ tidy tables
→ “measured 14×” claims
→ runtime numbers

…with no run artefact you can replay.

Same harness. Different job.

I’ve seen the failure mode up close: a corpus-level publishing decision got inferred from a stylistic argument (“don’t spam”). When challenged, the decision was reversed and a rule was hardened: publishing decisions about splitting/merging work must be explicit and logged with rationale.

The mistake is engaging the harness the same way in both contexts.

For code truth: demand small diffs and deterministic checks.

For publication truth: demand evidence anchors for claims (run identity + artefact pointer), and lint the prose to make missing evidence noisy.

Confidence is output. Truth is enforced.

Which truth surface in your harness is currently enforced — and which one are you still treating as optional?

#ClaudeCode #Trae #Qwen #AgentHarness #SoftwareEngineering
