I classified 63,053 records locally, at £0 API cost, in 87 hours.

One LLM call per record would have taken about 14 days. My window was five.

So I flipped the design. The LLM became the last resort, not the default. Every layer before it is cheaper and more deterministic than the next, and most records never reach the model at all.

That reuse waterfall is what turned a 14-day job into 87 hours on a MacBook.

63,053 records classified. 99.7% auto-accepted. £0 API cost. Zero data left the machine.

The non-obvious part was not a better prompt. It was everything around the model: reuse layers before inference, strict guards before reuse, and checkpointing so a crash does not delete a night's run.

One detail mattered more than any prompt tuning. Removing a single "reasoning" field from the JSON schema. Shorter outputs truncated less and triggered far fewer retry loops. One YAML key.

Question: what is the slowest, most expensive step in your pipeline that you are still treating as the default?

#AI #LLM #DataEngineering #MLOps #Python
