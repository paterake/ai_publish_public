Success wasn’t a feeling. It was a run you could defend.

That’s the shift that made a multi-agent build loop real: we stopped asking, “Does this look right?” and started asking, “Which run proves it?”

In building a local-first RAG platform, the useful role split ended up being simple:

- The harness (and its underlying models) acts as coordinator and judge: it plans the change, enforces constraints, and decides what counts as success.
- The local model acts as executor: it implements a bounded change and produces outputs we can inspect.

That only works if you make execution reviewable.

So we forced every meaningful iteration into the same loop:

Intent → contract (PRD) → bounded execution (thin runners) → evidence (run artefacts + telemetry) → review → success.

The practical breakthrough was consistency across assistants: different tools could review the same run evidence, spot gaps, make the code change, re-run locally, and keep repeating the loop until the outcome was defensible.

And it wasn’t used for a single feature. It was the build mechanism for the whole implementation stack: ingestion, hybrid retrieval, query orchestration, agentic retrieval, knowledge graph, API/UI surface, tool-call surface, HTTP agent, core LLMOps primitives, and visualisation/exploration.

The key detail is the stitching key: a `run_id`.

Without a correlation ID, you can’t join anything up. You end up with “we ran it” and a pile of outputs you can’t reliably trace or compare. With a `run_id`, the judgement step becomes mechanical: load the run’s artefacts, scan the scorecard, inspect anomalies, decide what to change next.

It’s not glamorous. It’s the difference between “AI helped me build something” and “this actually ships without drifting”.

Question: what would have to be true for you to trust the *next run* more than the last conversation?

#AI #LLMOps #SoftwareEngineering #ClaudeCode #Trae
