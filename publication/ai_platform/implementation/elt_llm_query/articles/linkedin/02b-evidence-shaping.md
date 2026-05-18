Top‑k is not a retrieval strategy.
More context often means worse answers.
So I shape evidence: fuse, rerank, dedupe, order.

The first time a RAG system returns a weak answer, most teams do the same thing:

They increase `top_k`.

Sometimes that helps. Often it makes answers worse: more context, less coherence, and a higher chance the model locks onto the wrong chunk.

📌 The turning point: treat retrieval as an engineering pipeline, not a parameter:
- sparse + dense retrieval (complementary failure modes)
- fusion (don’t pick one)
- reranking when it pays for itself
- context shaping (dedupe, diversify, order)
- answers always shipped with sources

Proof points to capture before publishing:
- [NEEDS_DATA: one before/after example where higher top_k made output worse]
- [NEEDS_DATA: one retrieval stage change and measured impact with run conditions]

Question: in your RAG system, are you still tuning “top_k and prompts” instead of shaping evidence?

#InformationRetrieval #RAG #DataEngineering #AI #Python
