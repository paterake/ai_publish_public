Most RAG quality problems start at ingestion.
If you destroy structure upstream, you can’t prompt it back.
So ingestion is treated as a substrate decision, not plumbing.

Most RAG “quality problems” are ingestion problems wearing a different name.

If you flatten layout-heavy documents, split tables mid-row, and embed everything as generic text, the system will feel unreliable no matter how much you tune prompts or rerankers.

📌 The turning point: treat ingestion as a substrate decision, not plumbing:
- Unstructured sources → chunk + embed → dual store (dense vectors + sparse docstores)
- Structured sources → deterministic JSON sidecars (facts retrieved precisely, not “searched”)

Why this matters:
- Better ingestion reduces hallucination risk because more facts are handled deterministically.
- Better chunk boundaries reduce “lost in the middle” failures because evidence remains coherent.
- Better substrate choice reduces cost because you retrieve less and prompt less.

Proof points to capture before publishing:
- [NEEDS_DATA: publish-safe before/after example of chunking a table]
- [NEEDS_DATA: cost/throughput delta from an ingest change]

Question: in your RAG system, which failures are you blaming on the model that actually started at ingestion?

#DataEngineering #RAG #AI #Python #MLOps
