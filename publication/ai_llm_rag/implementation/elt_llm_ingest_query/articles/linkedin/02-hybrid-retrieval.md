Why I stopped using single-retriever RAG

Dense-only retrieval works until it doesn’t.
Sparse sections get silently excluded and you get subtly wrong answers.
So I built hybrid retrieval and treated k as coverage, not quality.

Here’s the moment that made it obvious.

When I built the knowledge base layer for an internal governance handbook, I tried dense-vector-only retrieval first.

It worked well for paraphrases. Ask "what are the obligations around respect?" and it correctly found the relevant compliance section.

It failed completely on exact references. Ask about "Rule 43" or "CAS" and it returned semantically similar-but-wrong chunks. The rare, specific terms had no meaningful embedding neighbourhood.

So I added BM25 alongside dense vector retrieval: sparse for exact references, dense for paraphrase and concept search.

The two are fused via Reciprocal Rank Fusion — candidates that rank highly in both retrievers score highest. Then a reranker prunes the pool before the LLM sees anything.

RRF is a hedge:
it reduces retriever-specific blind spots instead of betting everything on one signal.

One thing I didn't expect to matter as much as it did:

The reranker's retrieve_k. I had it set to 24 (reasonable default). A specific query about regional obligations kept returning the wrong sections. Root cause: the correct section was short and sparse. Its chunks ranked 25th or lower — outside the window — and got silently excluded.

Raising to k=88 (2 sections × 44 collections) fixed it. Not a quality setting. A coverage setting.

📌 Low k doesn't produce errors, it produces subtly wrong answers.

Stack: local inference + ChromaDB + DocStore + LlamaIndex. 44 collections, ~1,726 chunks, zero cloud dependencies. Everything runs on a laptop.

Question: where are you silently excluding the right evidence — because a default k made it “someone else’s problem”?

#RAG #InformationRetrieval #DataEngineering #AI #Python
