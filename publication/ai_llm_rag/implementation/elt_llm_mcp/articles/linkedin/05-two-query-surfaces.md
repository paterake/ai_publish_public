Think of an AI assistant as someone walking into a library with a question.

The librarian doesn’t store the information. They know where everything is, what each source contains, and how to retrieve it usefully. They hold the catalogue. The books, the databases, the digital files — those are the information stores. The librarian decides what is findable. The content producers decide what exists. These are different responsibilities. And keeping them separate is the whole design.

That insight drove the architecture for how we exposed a governance knowledge base to two different audiences:

- Developers and IDE assistants get an MCP server — they call tools directly, chain queries, run pipelines.
- Business users get an HTTP agent — they ask questions in plain language and receive answers with a full evidence trail.

Both reach the same knowledge base. Neither duplicates the other.

Three things the system knows that a naive “ask the AI” approach does not:

- Not every question needs synthesis. If the answer is already extracted and stored, read the file. A catalogue lookup returns in milliseconds. RAG takes seconds and adds model variance. Route to the cheapest correct source first.

- Relationship questions need a map, not a library. “What governs this role?” is not a text retrieval question — it is a traversal question. The knowledge graph answers it deterministically. The language model explains what the graph returned.

- A fluent answer with no evidence trail is a demo, not a system. Every response carries the record of what was looked up, in what order, and what evidence it found. That is what makes it auditable — and trustworthy.

The tool contract shapes are stable regardless of domain:

`lookup_json_record` → returns the extracted record directly from the catalogue
`query_graph` → returns the governance chain: [“Role”, “…”, “the governing body”]
`query_rag` → returns answer + sources + confidence score: 0.87

📌 One orchestration layer. Two surfaces. The same behaviour for every client.

Question: when your AI assistant gets a question wrong, can you trace exactly what it looked up and why — or does it just give you a different answer on the next try?

#MCP #KnowledgeGraph #AIEngineering #DataEngineering #LLMOps
