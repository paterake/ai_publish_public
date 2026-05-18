What building with AI looks like when the data can’t leave the machine

The moment that changed the architecture wasn’t a model upgrade.
It was a constraint.

Enterprise data that cannot leave the machine means: no cloud LLMs, no managed APIs, no “just send it to a service”.
So I built the whole platform locally — on a laptop.

What surprised me is where the leverage actually was:
the win didn’t come from “better prompting”. It came from calling the LLM as rarely as possible, and forcing every AI step through quality gates.

This is a cost and reliability model:
every LLM call is latency, non-determinism, and a failure surface.

One proof point (real runs, not benchmarks):

63,053 records classified overnight. £0 API cost. 99.7% auto‑accepted.
That only works when the system is deterministic first and AI last — reuse before inference, gates before output.

📌 Treat the model as the most expensive, least reliable step in your pipeline — and design everything around avoiding it.

Question: what’s the constraint in your environment that “standard AI tooling” quietly assumes away?

#AI #AIEngineering #DataEngineering #ClaudeCode #Trae
