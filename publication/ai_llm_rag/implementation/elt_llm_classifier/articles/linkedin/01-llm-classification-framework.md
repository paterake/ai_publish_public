I classified 63,053 records locally at £0 API cost. The obvious approach would have taken 87 hours.

The moment I stopped “doing AI” and started doing engineering was a simple calculation:

63,000 records × ~5 seconds per LLM call = ~87 hours.

And that assumes zero crashes, perfect JSON, and a dataset you’re allowed to ship to an API.

So I flipped the design:
📌 make the LLM the last resort, not the default.

Think of it as a cascading cache:
each layer is cheaper and more deterministic than the next.

That’s how you get:

63,053 records classified. 99.7% auto‑accepted. £0 API cost. Zero data left the machine.

The non-obvious part wasn’t a better prompt. It was everything around the model:
reuse layers before inference, strict guards before reuse, and checkpointing so a crash doesn’t delete a night’s run.

One tiny detail made the difference between “works” and “runs overnight”:
removing a single “reasoning” field from the JSON schema produced a 14× throughput improvement because it slashed truncation and retry loops. One YAML key change.

Question: what’s the slowest, most expensive step in your pipeline that you’re still treating as the default?

#AI #LLM #DataEngineering #MLOps #Python
