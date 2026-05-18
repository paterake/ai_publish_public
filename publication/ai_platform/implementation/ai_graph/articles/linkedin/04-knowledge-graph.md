RAG is bad at relationships.
If the question is a path, synthesis is the wrong substrate.
So I assembled a typed graph from audited catalogs.

RAG is good at answering questions over text.

It’s bad at answering questions over relationships.

If your question is “what governs what?”, “what is upstream of X?”, or “show me the chain from A to B”, a synthesis model is doing the wrong job. You need traversal semantics, not more prompt engineering.

What we built instead: a typed knowledge graph assembled deterministically from structured catalogs (entities, relationships, reference lists), exported in standard formats for query and visualisation.

Key turning points:
- We kept raising retrieval `top_k` to “find the missing edge”. The chain was still incomplete because the relationship wasn’t a paragraph anywhere — it was a shape across sources.
- The LLM sometimes filled in plausible edges when evidence was incomplete. That’s acceptable for summaries, not for governance relationships.
- 📌 Once we assembled a directed graph from audited outputs, governance chains became deterministic 1-hop/2-hop traversals, not 10-chunk syntheses.

Proof points to capture before publishing:
- 894 nodes, 772 edges (2026-04; Apple M3 Pro MacBook, 18GB RAM; internal governance handbook + enterprise architecture model)
- [NEEDS_DATA: one publish-safe example of a governance chain query result]

Question: where in your systems are you still using “search + synthesis” to answer what is really a graph traversal problem?

#KnowledgeGraph #DataEngineering #AI #RAG #Python
