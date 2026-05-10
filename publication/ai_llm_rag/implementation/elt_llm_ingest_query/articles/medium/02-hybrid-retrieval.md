# Building an Enterprise Knowledge Base from Governance Documents

Dense-only retrieval works until it doesn’t. Sparse sections get silently excluded and you get subtly wrong answers. This is the architecture of a hybrid retrieval pipeline that treats `k` as a coverage metric rather than a quality metric, preventing exact-match entity failure.

## Key takeaways

- Hybrid retrieval works because failure modes are complementary: BM25 for exact references, dense vectors for paraphrases, then fusion and reranking for precision.
- Treat `k` as a coverage control, not a quality knob: too low and sections silently drop out, producing plausible but wrong answers.
- Ingestion decisions surface later as “RAG quality”: structured PDF extraction, table-aware chunking, and tokenisation edge cases can make or break retrieval.

---

## Who this is for

You are building a RAG system over a long, structured document and you care about reliability (citations, coverage) more than demos.

## What you’ll learn

- Why hybrid retrieval works in practice (and what each retriever misses)
- How ingestion decisions (PDF parsing, table handling) show up later as “RAG quality”
- Why reranker coverage is a silent failure mode (and how to reason about it)
- Why query expansion mode and context ordering shape what the LLM ultimately synthesises

## Constraints

- Local-first operation with zero data egress
- A document long enough that full-context injection is not viable

## Why the naive approach fails
A 763-page governance handbook. Dozens of policy sections. Hundreds of defined terms. The standard approach would be to dump it into an LLM's context window and let it answer questions directly. That breaks down quickly: context windows overflow, answers become unreliable for sparse sections, and there is no audit trail for which passage drove a given answer.

The better framing: build a queryable knowledge base from the document. Something that can retrieve the right section for any question, synthesise a structured answer from authoritative passages, and remain inspectable. That is RAG — Retrieval-Augmented Generation.

This article walks through the specific design decisions behind the ingestion and retrieval layers of a local-first RAG system built for an internal governance handbook. Everything runs locally. No data leaves the machine.

---

## Why RAG at All?

There are two simpler alternatives, and both were considered seriously before settling on RAG.

**Option 1: Full-document context injection.** Pass the entire handbook to the LLM on every query. Fails at scale: a 763-page document exceeds safe context limits for useful synthesis, and quality degrades badly in the middle of very long contexts.

**Option 2: Manual chunking with keyword search.** Split the document by section and use keyword matching. Works for exact lookups but misses paraphrases entirely. Ask "what are the obligations around respect?" and BM25 returns nothing useful if the section header says "Respect Code" but your query says "obligations around respect".

RAG solves both: chunk intelligently, embed semantically, retrieve broadly, rerank precisely.

**A third option was considered and ruled out: agentic RAG at the ingestion layer.** An agentic
loop — where the LLM drives iterative retrieval, deciding what to fetch next based on what it
has so far — adds flexibility for sparse or structurally ambiguous entities. But the governance
handbook has inferrable structure: numbered sections, consistent heading hierarchy, defined
terms with explicit glossary entries. When structure is explicit in the source, standard
retrieval can exploit it directly without LLM-driven search loops. There is also an idempotency
argument: LLM inference at ingestion means the same document re-ingested could produce different
chunks, metadata, or boundaries depending on model state. Ingestion must be repeatable and
fail-loudly — deterministic parsing guarantees that.

Agentic retrieval becomes necessary when the entity being queried is sparse or structurally
implicit — which is precisely the problem Article 3 addresses. The boundary between standard
RAG (Article 2) and agentic RAG (Article 3) is not arbitrary: it is drawn at the point where
the structure of the source document stops being exploitable by deterministic means.

---

## Ingestion: Getting the Chunks Right

The governance handbook arrives as a PDF. The first engineering challenge is extracting structured text from it without mangling tables, headers, and numbered paragraphs.

### Docling for Structured PDF Extraction

Getting structured, reliable text out of a governance PDF was not a one-step decision. Three tools were tried in sequence, each rejected when retrieval quality exposed the failure.

**Iteration 1 — pypdf.** Standard linear text extraction. Fast and simple. The definitions section — a large table of terms and meanings — collapsed into continuous text: header, definition, and clause markers merged into a single unbroken stream. Downstream enrichment pipelines running against chunks built from this extraction returned fragments that looked plausible but contained merged or mis-ordered content. Entity definitions sourced from that section were unreliable.

**Iteration 2 — PyMuPDF4LLM.** Better Markdown-style output, significantly faster. Large tables still fragmented across chunks — cell content split mid-row for dense sections. The problem was less severe but not solved. The definitions table still could not be used reliably as a retrieval source.

**Iteration 3 — Docling.** Layout-aware, deep-learning-based parsing. Table structure preserved as pipe-delimited Markdown. Section boundaries detected structurally. A formal comparison was run across all three tools specifically measuring definition table preservation — Docling was selected.

The trade-off accepted: Docling parses a full handbook in ~250s versus seconds for the alternatives. This is mitigated by the section-split cache — Docling runs once per source version; subsequent re-ingestions skip conversion entirely.

```yaml
preprocessor:
  type: DoclingPreprocessor
  section_cache_dir: <cache_dir>
```

The cache check runs first on every pipeline start. If the cache exists, Docling is skipped entirely. The extraction cost is paid once per source version.

This is the cache-at-the-boundary principle applied consistently across the platform: every
expensive operation at an external boundary — PDF parsing, embedding calls, LLM synthesis —
pays its cost exactly once per unique input. Docling is the clearest example because the cost
is visible (minutes, not milliseconds), but the same discipline applies to embeddings: chunks
are embedded once at ingestion and stored in ChromaDB; they are never re-embedded at query time.
The boundary is where variance and cost enter the system. Caching there eliminates repeated
cost at no quality penalty.

### Splitting: Prose and Tables Are Different Problems

Standard sentence splitters treat all text the same. That works for paragraphs but fails badly for tables. A governance document contains dozens of rule tables — structured grids of obligations, categories, and sanctions. Splitting a table by sentence produces chunks that lose all column context.

`TableAwareSentenceSplitter` handles them separately:

- **Prose**: standard sentence splitting at `chunk_size=256 tokens`, `chunk_overlap=32`
- **Tables**: each row becomes its own chunk at `chunk_size=512`; header row is prepended to every data-row chunk for context

The overlap value is deliberate. Without overlap, a sentence split at a chunk boundary loses
context on both sides: the chunk ending mid-clause has no resolution, and the chunk starting
mid-thought has no setup. At `chunk_overlap=32`, enough tokens carry across the boundary to
preserve a defined term or a clause reference — the things that matter most in governance text.
32 was chosen as the minimum overlap that eliminated boundary-split failures in testing without
meaningfully increasing chunk count or storage.

There is one critical edge case: the Markdown separator row (`|---|---|`). Every character in that row tokenises separately under BERT tokenisation. A row with 10 columns produces a chunk that is almost entirely separator characters — each one a token. Passed to Ollama's embedding API, it triggers an `HTTP 400` (chunk too long, token limit exceeded). The splitter explicitly detects and skips separator rows:

```python
def _is_separator_row(self, line: str) -> bool:
    return bool(re.match(r'^\|[\s\-|]+\|$', line.strip()))
```

Skipping this was not obvious. The failure mode (HTTP 400 from the embedding endpoint) looks like a network error at first glance. The root cause is tokenisation, not the request itself.

### What Goes Where

After splitting, chunks are stored in two parallel stores:

| Store | Purpose |
|-------|---------|
| **ChromaDB** | Dense vector retrieval — 44 collections, one per document section |
| **DocStore** (SimpleDocumentStore) | Sparse/BM25 retrieval — same chunks, different index |

The final result: **44 collections, approximately 1,726 chunks** across the full handbook.

---

## Query Expansion: One Query or Many?

Before retrieval runs, the pipeline decides how many query variants to use.

For **batch runs** — the catalog enrichment pipeline covered in Article 3 — `num_queries=1`. The
query passes through as-is. This is deliberate: batch runs process hundreds of entities and must
be deterministic and reproducible. LLM-generated query variants introduce model-state dependency
— the same entity re-run could expand differently, producing different retrieval paths and
different enrichment outputs. Reproducibility is not optional in a quality-gated pipeline.

For **interactive queries** — the agent and API surfaces built on this pipeline — `num_queries=3`.
The LLM generates two additional phrasings before retrieval runs. A user asking about a concept
in one form benefits from variant phrasings that may match sections the original query misses.
In an interactive context, the marginal non-determinism is acceptable; in a batch catalog run,
it is not.

The distinction is not cosmetic. Every downstream consumer of this pipeline inherits one of
these two modes, and the choice shapes whether outputs are reproducible.

---

## Why Hybrid Retrieval?

Once chunks are stored, the retrieval question is: how do you find the right ones?

Dense vector search works by embedding the query and measuring cosine similarity against stored chunk embeddings. It handles paraphrases and semantic similarity well. It struggles with exact term matching — rule references, specific numbered clauses, defined terms.

BM25 is classic term frequency retrieval. It handles exact matches and rare terms well. It fails completely on paraphrases: if the query uses a synonym, BM25 returns nothing.

Here is what that looks like in practice. Ask for an exact reference (“Rule 43”). Dense-only retrieval returns a thematically related clause, not the numbered rule. The answer reads plausibly, but the cited passage is wrong — and unless you inspect citations, you will not notice.

These failure modes are complementary, which is why hybrid retrieval works.

**What BM25 catches that dense vector misses:**
- Exact rule references: "Rule 43", "Section 7.3"
- Rare defined acronyms: short programme names that appear infrequently (no semantic neighbourhood)
- Table entries: structured cells with specific values

**What dense vector catches that BM25 misses:**
- Paraphrases: "obligations around respect" → retrieves "Respect Code" section
- Topic synonyms: "financial fair play" → retrieves "sustainability regulations"
- Questions about concepts not named in the chunk: "what happens if a department overspends?" → retrieves budget rule section without matching "overspend"

### Reciprocal Rank Fusion

The two retrievers are fused using Reciprocal Rank Fusion (RRF). Each retriever ranks its results independently. RRF combines the rankings:

```
RRF_score(d) = Σ 1 / (k + rank_i(d))
```

where `k=60` dampens the influence of very high ranks. A document that appears in the top 10 of both retrievers scores higher than one that appears at rank 1 in only one. This rewards consistent relevance across retrieval methods, not just peak performance in one.

---

## The Reranker: From Broad Retrieval to Precise Selection

After fusion, the combined candidate set is larger than what should be sent to the LLM. The reranker's job is to score each candidate for relevance to the original query and select the top K.

Two reranker options were evaluated:

| Approach | How It Works | Trade-off |
|----------|-------------|-----------|
| **Embedding cosine similarity** | Re-embed query + chunk, compute cosine | Fast, same embedding model, symmetric |
| **Cross-encoder** | Fine-tuned model scores query+chunk jointly | More accurate, separate model, slower |

The choice was embedding cosine. Reasons:
1. The same local embedding model is already running. No additional model to manage.
2. Cross-encoders require separate deployment and add additional latency to the retrieval path.
3. For governance text with long, well-structured sections, embedding cosine proved sufficient in testing.

### The k=24 Failure

The first production configuration used `reranker_retrieve_k=24` — retrieve 24 candidates for the reranker to score.

A specific query exposed the problem: queries about regional obligations kept returning generic rules instead of the correct short compliance section.

Root cause: the correct section is short and densely formatted. Its chunks had lower BM25 frequency scores (few of the query terms appeared frequently within the section) and marginally lower vector similarity scores than longer, more verbose sections. At k=24, the correct chunks consistently ranked 25th or lower — just outside the window.

Raising to `reranker_retrieve_k=88` fixed it. The rationale: 44 collections × 2 sections per collection average = ~88 candidates needed to guarantee at least one representative from each section reaches the reranker.

The lesson: reranker retrieve_k is not a quality knob — it is a coverage knob. Setting it too low silently excludes sparse but correct sections. You will not see an error; you will see wrong answers.

### Context Ordering: Where Chunks Land in the Window

After reranking, there is one more step before chunks reach the LLM: reordering them within
the context window.

LLMs do not attend uniformly across their context. Empirically, models attend better to content
at the start and end of the context window than to content in the middle — the "lost-in-the-middle"
effect. A chunk ranked second by the reranker that lands mid-context in a 32k window may
contribute less to synthesis than its score implies.

The fix is mechanical: place the highest-scored chunks at the start and end of the assembled
context, lower-scored chunks in the middle. The reranker ordering is preserved for selection;
only the assembly order changes for attention.

```yaml
llm:
  use_lost_in_middle: true
```

For governance text — long sections, multi-passage synthesis tasks — this reordering measurably
improves answer coherence for queries where the most relevant evidence would otherwise land
mid-context.

---

## LLM Synthesis

After reranking, the top K chunks are passed to the LLM with a structured synthesis prompt. The LLM's job is not to retrieve — that is done. Its job is to synthesise a coherent, cited answer from the provided passages.

Key synthesis config:

```yaml
llm:
  model: "<local_llm_model>"
  context_window: 32768
  num_predict: 700
  thinking: false
```

`thinking: false` is critical for models that emit internal reasoning blocks by default. Without it, the model prefixes its response with extended `<think>` blocks. Those blocks consume `num_predict` tokens. The actual answer never arrives — the caller returns "Empty Response". This is not a timeout; it is a budget exhaustion at the generation level.

The synthesis prompt always includes two glossary sections as preamble. Prepending them ensures the LLM has authoritative definitions for terms in scope, regardless of which sections the retriever returns.

---

## Results

| Metric | Value |
|--------|-------|
| Collections | 44 |
| Total chunks | ~1,726 |
| Ingestion time | ~250s (Docling, cached after first run) |
| Embedding model | local embeddings (on-device) |
| Synthesis model | local inference (on-device) |
| Context window | 32,768 tokens |
| Data egress | None |

The system handles queries across the full governance document, with section-level attribution for every answer.

---

## Architecture Decisions Summary

| Decision | Alternative Considered | Why This |
|----------|----------------------|---------|
| Docling for PDF extraction | pypdf, PyMuPDF4LLM | Layout-aware parsing preserves table structure; pypdf/PyMuPDF fragmented definition tables |
| TableAwareSentenceSplitter | Standard sentence splitter | Tables need row-level chunking, not sentence splitting |
| chunk_overlap=32 | No overlap | Overlap carries defined terms and clause references across chunk boundaries |
| Hybrid BM25 + dense vector | Dense-only | BM25 covers exact terms/rules that vector misses |
| RRF fusion | Score averaging | Rewards consistent relevance, not single-retriever peak |
| Embedding cosine reranker | Cross-encoder | Same model already running; cross-encoder adds deployment cost |
| reranker_retrieve_k=88 | k=24 (initial) | k=24 silently excluded sparse sections (found via Section 40 failure) |
| num_queries=1 (batch) / 3 (interactive) | Fixed single query | Batch requires determinism and reproducibility; interactive trades it for broader recall |
| Lost-in-middle context reorder | Preserve reranker order | LLMs attend better at context boundaries; reordering places best evidence where the model uses it |
| Local embedding + synthesis | Cloud APIs | Fully local, no API cost, no data egress |
| 44 ChromaDB collections (one per section) | Single collection | Per-section indexing enables section-targeted retrieval |

---

## What This Enables

This infrastructure layer is the shared substrate for three downstream consumers: the
quality-gated agentic retriever (Article 3), the MCP tool surface, and the HTTP agent interface.
All three query the same hybrid index, use the same reranking pipeline, and inherit the same
context ordering. Getting this layer right is what makes everything built on top of it reliable.

The retrieval pipeline produces chunks accurate enough that the quality gate (Article 3) rejects
only ~15–30% of entities on the first pass. The hybrid approach means exact rule references,
semantic paraphrases, and table-structured data all retrieve correctly from the same index.
The batch/interactive query expansion split means the catalog pipeline is deterministic and
the agent surfaces are responsive — from the same underlying engine.

The full stack runs locally on a laptop. No cloud. No API key. No data egress.

---

## Limits / when not to use

- `reranker_retrieve_k` is a coverage knob; the right value depends on corpus structure and chunking strategy.
- PDF parsing and table handling are workload-dependent; layout-heavy sources can still require manual fixes.
- Hybrid retrieval improves recall, but it does not guarantee truth; you still need sources and explicit “not found” behaviour.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; 44 collections (~1,726 chunks); `reranker_retrieve_k=88`; context window 32k; local inference
- Dataset class: internal governance handbook (anonymised; no client identifiers)
- Non-reproducible from this article: exact prompts and internal contracts

---

*The code for this system is part of a local-first AI platform for enterprise governance knowledge. The next article covers the quality-gated agentic retrieval layer built on top of this infrastructure.*
