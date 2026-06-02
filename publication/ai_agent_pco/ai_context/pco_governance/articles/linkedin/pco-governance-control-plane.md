I have been coding for nearly four decades. Basic, Assembly, COBOL, Java, Scala, Python.

Vibe coding didn't kill my desire to code. It accelerated my ability to deliver more.

What I didn't see coming was the governance problem.

Every new session, every tool switch, I was re-establishing the same baseline. Keep solutions modular. Drive everything through configuration. Build on libraries, not from scratch. Leave a proper audit trail.

These aren't preferences. They're the defaults of someone who's watched what happens to systems that skip them.

I moved the rules into projects. That worked until I had multiple projects, copied the controls across each, enhanced them separately, and watched them drift. One had tighter security posture. Another had better operational bounds. Neither had the best of both.

That's a control plane problem, not a prompting problem.

As a data engineer, you recognise this pattern and refuse it. You centralise the canonical version. Every consumer inherits from it.

That is what I built. One governance layer, pushed into every governed repo as a dependency. I call it the PCO: Policy, Control plane, Observability.

Policy: what's allowed and who decides. Control plane: deterministic routing, bounded tools, stop and deny points. Observability: run evidence, lineage, audits, exception and approval records.

Two things made it real.

Rules are shared across assistants. Claude Code, Trae, Qwen. Each tool has a thin shim pointing at one shared ruleset. Switching tools stopped being a reset.

Policy violations produce a structured failure signal. Not "be careful." A denial with evidence:

Action denied by governance policy.
Reason: Protected change requires approval at this risk tier.
Next: request approval or scope down the change.

Roll out AI coding to twenty engineers without shared governance and each one defines good differently. Code from one team won't couple cleanly to a second. You won't know for months, by which point the integration work costs more than the original builds.

One governance layer, running before the first prompt lands.

In Claude Code, the rules file tells the model what to do. It doesn't stop anything. The PCO's job is to make rules stop things.

The contract is what you can share: what gets enforced and what evidence each run produces. The implementation stays private. Evidence is what makes it credible to publish.

When inconsistency shows up in AI-assisted delivery across your team, what's the first thing you look at — the tool or the absence of a shared baseline?

#AI #SoftwareEngineering #DeveloperTools #Governance #SoftwareArchitecture
