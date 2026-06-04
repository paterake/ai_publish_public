The data engineering instinct is to centralise.

When you see the same pattern across three pipelines, you do not update three config files — you create one canonical schema. Every consumer inherits from it.

That instinct is correct in AI systems. It just arrives a module or two too late.

Three AI modules. Three config schemas. Three slightly different Ollama wiring blocks.
Then the embedding model changes — and the update has to happen in three places.

That's the duplication trap. It shows up before you realise it, and it compounds.

So across a ten-module AI platform, I built a shared primitives layer instead.
One config schema. One provider wiring surface. One baseline profile. Every module
overrides only what it changes.

Scope note: this isn’t an “agent framework” replacement. It’s the plumbing layer underneath your orchestration library: shared config + provider wiring so every module connects to models, embeddings, and retrieval defaults the same way.

It also keeps change local: extend the wiring once (for example, add a provider), and every module inherits the capability without rework.

This isn't just a DRY decision — it's a token economy one. Every token a coding session spends re-deriving infrastructure is a token the domain problem never gets. Stop burning context budget on plumbing.

CoreConfig — one shared, code-blind configuration schema (vector store, model wiring, chunking, query).
Every module loads the same schema. No module defines its own.

The baseline + profile system — defaults live in five split YAML files, one concern
each. A module override specifies only what differs; the loader deep-merges the rest.
A minimal module profile is 6 lines. The result is a complete, deterministic config.

Provider portability — the portable LLM call surface (LLMCaller) takes a provider
via environment variable: AI_AGENT_CORE_PROVIDER=azure_openai. Module code doesn't import
a provider SDK. Switching from local Ollama to a hosted API is one environment change.
No code changes. Telemetry stays wired. Guardrails stay wired.

Config-driven CLI — a shared runner_args.yaml spec builds argparse contracts
declaratively. Treat the CLI help output as authoritative; don’t maintain a separate captured-help doc that can drift.

The principle underneath all of this: any value that changes when you change providers,
models, or workloads belongs in configuration, not code. Keep the mechanism generic.
Keep the domain context in YAML.

The full piece goes deeper: the token economy argument in full, each primitive with implementation detail, the structural fix that enforces the shared layer at import time, and the 13-module platform it underpins.

Question: in your AI stack, how many places would you need to change if the embedding model changed tomorrow?

#SoftwareEngineering #Python #RAG #AIEngineering #LLMOps
