Single-pass RAG failed for ~50% of my entities. Here's what replaced it.

Single-pass RAG loves boilerplate.
For sparse entities, fluency replaces usefulness.
So I added a quality gate and an agentic fallback loop.

Here’s the kind of output that forced the redesign.

I was enriching 175 entities from an enterprise architecture model with definitions and rules from a 763-page handbook.

Standard RAG — retrieve, synthesise, done — returned this for roughly half of them:

"This is an important concept in enterprise governance. It relates to the organisation's operational framework and compliance requirements."

Technically coherent. Zero information content. Indistinguishable from a confident hallucination.

The fix was to stop treating synthesis as the end of the pipeline.

After every answer, I score it. If it’s too short, generic, or missing actual rules, it fails.
Only then does an agentic fallback kick in: refine the query, bypass embeddings with keyword search when needed, and retry until the gate passes (or a strict iteration cap is hit).

📌 The key is that “quality” is explicit: the gate penalises boilerplate and rewards evidence density.

**Real example — "Market Segment":**
1. RETRIEVE → returns commercial sections, no formal definition. Score: 0.31.
2. KEYWORD → finds the term in a broadcast rights clause.
3. RETRIEVE (targeted) → pulls the specific clause + context.
Score: 0.67. Gate passes. Done in 3 iterations.

**Results across 175 entities:**
- 169/175 ended up with both a formal definition AND governance rules
- Only 17 needed the agentic loop
- Average 1.1 iterations for those 17
- 6 correctly flagged as "not found" (they genuinely don't appear in the handbook)

The 6 "not found" entries are the honest result. The handbook doesn't attempt to define every model abstraction — and surfacing that gap is more useful than filling it with boilerplate.

No cloud. No API keys. Runs locally on a laptop.

Question: where in your RAG pipeline are you still shipping “technically fluent, zero-information” outputs because nothing is scoring quality before it reaches users?

#RAG #LLMOps #DataEngineering #AI #Python
