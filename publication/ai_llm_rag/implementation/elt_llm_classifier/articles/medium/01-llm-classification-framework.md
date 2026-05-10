# The Wrong Way to Use AI for Classification — and What to Build Instead

Calling an LLM for every record is the obvious approach to bulk text classification. It is also the wrong one. Here is the design that classifies 63,000 records locally, at zero API cost — and the non-obvious failures that nearly derailed it.

## Key takeaways

- A reuse-first waterfall (exact → fingerprint → fuzzy → semantic → cluster → LLM) cuts LLM calls without guessing.
- Resume semantics are a feature: checkpoints and deterministic row keys make multi-day runs safe to stop and restart.
- Prompt and output budgets matter: removing unnecessary fields and capping bulk output produced a measured 14× speed-up on the LLM-only path.

---

## Who this is for

You are building a bulk classifier over messy text and you have already tried “LLM per record” (or you are about to).

## What you’ll learn

- Why “deterministic first, AI last” is an architecture choice, not an optimisation
- How a reuse-first waterfall reduces LLM calls without turning quality into guesswork
- Which failure modes only appear at scale (and what fixes actually held)

## Constraints

- Local-first operation (privacy and governance constraints assumed)
- Large batch backfills (resume semantics matter more than elegance)

## Why LLM-per-record fails at scale

Most organisations accumulate text-heavy records — tickets, incidents, emails, surveys — that need to be categorised into a controlled taxonomy for routing, analytics, and compliance. Manual triage doesn't scale. Rules-based systems break as language drifts. So the obvious move is: call an AI model for every record.

Here is what that actually looks like at scale.

A 10-year incident backlog: **63,053 records**. At five seconds per LLM call — a realistic estimate for a local 7B model — that is **87 hours of uninterrupted processing**. And that assumes: no crashes, no timeouts, no retries, no restarts. It also assumes the data can leave the machine. In organisations with sensitive operational data — incident descriptions, system names, user IDs — it often can't.

The naive approach fails at scale not because AI is bad at classification. It fails because calling AI for *every* record is architecturally wrong.

---

## The Failure of Obvious Approaches

Before describing what works, it's worth being honest about what doesn't.

**Naive LLM pipeline:** Call the model for every row. Works for 100 records. Crawls at 10,000. Breaks overnight at 63,000. Any crash loses progress. No way to resume. No way to audit whether the model made a consistent decision about the same text twice.

**Simple caching:** Cache LLM responses keyed on exact text. Works for perfectly identical tickets. Useless for real datasets where the same incident is described in ten different ways ("server is down", "srv not responding", "cannot connect to srv — urgent"), none of which share enough characters to match.

**Fuzzy matching only:** Works for near-duplicate text. Fails when two tickets describe the same problem in completely different words — same meaning, zero shared vocabulary. Fuzzy string matching cannot bridge that gap.

**The gap:** No single strategy is sufficient. Exact matching is too strict. Fuzzy matching is too shallow. LLMs are too slow and too expensive for every record. The solution is not to choose one — it is to layer all of them, in the right order, with strict quality gates at each layer.

---

## The Insight: Deterministic First, AI Last

The key design principle is not "use AI better." It is: **use AI as little as possible**.

Every time a decision can be made deterministically — same result every time, instantly, with no possibility of hallucination — it should be. The LLM is called only when every deterministic and statistical shortcut has been exhausted.

This sounds obvious in retrospect. In practice, most AI classification pipelines are built the other way around: AI first, with occasional caching bolted on later as an optimisation. Inverting the order changes the architecture entirely.

---

## The Architecture: A Six-Layer Reuse Waterfall

For each record, the classifier evaluates six strategies in order — cheapest and most deterministic first, LLM last:

```
1. EXACT MATCH CACHE       — identical text, same taxonomy choices → instant, 100% safe
2. FINGERPRINT CACHE       — normalised near-duplicate hash → instant, no hallucination
3. FUZZY STRING MATCH      — Jaro-Winkler similarity → fast (~1ms), catches typos/minor edits
4. SEMANTIC CACHE          — embedding cosine similarity → ~30ms, catches same meaning
5. CLUSTER REUSE           — centroid cosine similarity → ~30ms, catches recurring patterns
6. LLM FALLBACK            — local inference, no data leaves the machine
```

The order is not arbitrary. Each layer covers a class of similarity that the layer above cannot. Fuzzy matching catches near-identical text; semantic matching catches near-identical *meaning* — cases where two tickets share no words but describe the same problem. Fuzzy sits before semantic because it is two orders of magnitude cheaper. Semantic sits before LLM because even at ~30ms per embedding call, it is 100× faster than a generation call.

**Reuse is permitted only when all guards pass:**
- The taxonomy version fingerprint matches (prevents reusing decisions from a previous taxonomy revision)
- The similarity score exceeds a strict threshold (0.975+ for semantic; 0.96+ for fuzzy)
- The prior result was auto-accepted (not flagged for human review)
- The prior confidence score exceeds the minimum threshold

If any guard fails, the pipeline falls through to the next layer. There is no silent degradation.

---

## The Non-Obvious Engineering Problems

Building the pipeline is straightforward. Making it survive 63,000 records, multiple crashes, and 80 hours of continuous operation is where the real engineering happened. Here are the failures that were only visible at scale.

### The Python Falsy Bug That Silenced All Caches

After the first restart of the live run, the semantic and cluster caches were showing zero entries on disk — even though thousands of rows had been processed.

The root cause: four lines in the classifier all looked like this:

```python
if self._semantic_cache and self._embedder and result.get("status") == "auto_accepted":
    # seed the cache
```

This is idiomatic Python. It is also wrong. Python evaluates `bool(obj)` by calling `obj.__len__()`. An empty semantic cache has `len() == 0`, so `bool(empty_cache)` is `False`. The cache object existed, was initialised correctly — but was treated as falsy every single time. All four seeding conditions were skipped from row 1 of every run.

The fix is four characters: `is not None`.

```python
if self._semantic_cache is not None and self._embedder is not None and ...:
```

This is the kind of bug that never appears in unit tests, because test fixtures rarely use genuinely empty objects with `__len__` defined. It only surfaces when you run 10,000 rows and notice the cache file is still 0 bytes.

**The lesson:** In Python, `if obj:` and `if obj is not None:` are not interchangeable. Any object that implements `__len__` will evaluate as falsy when empty. Cache-like objects, index-like objects, and collection-like objects all have this property.

### The Embedder That Permanently Disabled Itself

The original `OllamaEmbedder` implementation had a simple failure handler:

```python
except Exception:
    self._disabled = True
```

One transient Ollama timeout — the kind that happens when the LLM model and the embedding model compete for the same process resources — and the embedder was permanently disabled for the rest of the run. All semantic and cluster reuse silently stopped. The system continued, still classifying correctly via the LLM, with no error — just a permanent, silent performance regression.

The fix replaces the permanent disable with a circuit breaker: 2 retries at 0.5s flat delay, trips after 10 consecutive failures, auto-recovers on the first success. The 0.5s delay is not arbitrary — it is calibrated to Ollama's typical recovery time after resource contention, while keeping worst-case retry overhead at ~10% of a typical LLM call (0.5s vs ~5s).

There is also a secondary effect: when the embedder is disabled by the circuit breaker, Ollama's resources are no longer split between two simultaneous model calls. The LLM gets the full GPU allocation. This produced a measurable throughput improvement for LLM-only rows even when the embedding path was inactive.

### The Crash-Resume Duplicate Problem (and the Fix)

The output format is JSONL — an appendable, line-delimited file rather than a database write
or in-memory accumulator. This is a deliberate engineering choice for pipelines that run for
hours or days: a crash at row 40,000 of 63,000 loses nothing already written. The file is
not rewritten on resume — new results are appended, and a checkpoint tracks where processing
left off.

A crash mid-batch leaves a structural gap: the JSONL output file has been partially written, but the checkpoint has not been saved yet (it saves at the end of each batch). If you resume naïvely, you can re-process and re-append those rows — producing duplicates.

The fix implemented here is a pragmatic reconciliation step: on resume, the runner scans the existing results file for `row_md5` values and adds any missing ones into the checkpoint before processing continues. That makes resume safe without requiring a full rewrite of the output protocol.

**Lesson for anyone building resumable batch pipelines:** the checkpoint save and the output write must be atomic. If they are not, crash-resume will produce duplicates. Deduplication by a deterministic row key is the safe recovery procedure.

### The 14× Throughput Improvement From One Config Change

The prompt schema originally included a `reasoning` field — a short explanation of why the model chose the category. This sounds useful for auditing. In practice, it caused two problems:

1. The model sometimes used the `reasoning` field to write paragraphs, exhausting the `num_predict` budget before the JSON structure closed — producing invalid output.
2. The generation length was 2–4× longer than necessary, directly reducing rows/second.

Removing `reasoning` from the bulk run prompt schema, and lowering `bulk_num_predict` to 128, produced a **14× throughput improvement** on LLM-only rows — measured on the live dataset.

This is the counterintuitive result: *reducing* what you ask the model to produce *improves* classification reliability, not just speed, because shorter outputs are less likely to truncate mid-JSON.

---

## What the run proved

These are measurements from a live enterprise incident backlog:

| Metric | Value |
|--------|-------|
| Output records written | 63,053 (`reclassified.jsonl`) |
| Review queue size | 206 (`review_queue.jsonl`) |
| API cost | £0 — 100% local inference |
| Data leaves the machine | Never |
| Reclassification batch processed | 44,803 rows |
| Total elapsed (reclassification batch) | ~67.3 hours (242,437s) |
| Overall throughput (reclassification batch) | 0.185 rows/sec |
| Throughput at run start (LLM-only) | ~0.20 rows/sec |
| Throughput as caches warm | ~0.71 rows/sec (3.5× improvement) |
| Throughput improvement from removing reasoning field | 14× on LLM-only path |

**Strategy distribution** (44,803-record reclassification batch):

| Strategy | Records | Share |
|----------|---------|-------|
| LLM fallback | 25,346 | 56.6% |
| Fingerprint cache | 6,673 | 14.9% |
| Semantic cache | 5,297 | 11.8% |
| Fuzzy match | 4,620 | 10.3% |
| Cluster reuse | 2,867 | 6.4% |

The LLM handled the majority because this was a reclassification run against a new taxonomy — the caches were cold at start. In a steady-state run against an existing taxonomy, the fingerprint, semantic, and fuzzy layers handle a much larger share.

The throughput improvement from cache warming is not linear. The first 10% of a run is the slowest — caches are empty, every row hits the LLM. As the fingerprint cache fills with the most common ticket templates, and the semantic cache builds a vocabulary of normalised problem types, the hit rate climbs and the LLM handles a shrinking fraction of rows.

**End-to-end examples (major paths):**

The raw outputs record `match_confidence` when a prior classification was reused. In this run:
- `match_confidence = 1.0` indicates deterministic reuse (exact/fingerprint).
- `0.96 ≤ match_confidence < 0.975` indicates fuzzy string reuse.
- `0.975 ≤ match_confidence < 0.99` indicates embedding-based reuse (semantic/cluster).
- No `match_confidence` indicates the LLM path was used (no safe reuse).

Each example below is taken from the real output, with ticket text scrubbed and domain nouns abstracted.

Fingerprint/exact reuse:

```json
{
  "ticket_text_sanitised": "User contacted as require password reset for internal portal. User is unable to access internal portal and password reset yesterday but still not logging in.",
  "match_confidence": 1.0,
  "confidence_score": 0.9,
  "classification_status": "auto_accepted",
  "category_id": "subcat_non_issue_user_guidance_how_to_help",
  "category_name": "Non-Issue > User Guidance > How-To / Help"
}
```

Fuzzy reuse (near-duplicate):

```json
{
  "ticket_text_sanitised": "Please remove the invoice and the associated registration record.",
  "match_confidence": 0.960919540229885,
  "confidence_score": 0.9,
  "classification_status": "auto_accepted",
  "category_id": "subcat_data_issues_mapping_incorrect_field_mapping",
  "category_name": "Data Issues > Mapping > Incorrect Field Mapping: Field mapped incorrectly"
}
```

Semantic/cluster reuse (meaning-level):

```json
{
  "ticket_text_sanitised": "Unable to access a third-party platform: service unavailable. [URL] [TICKET_ID]",
  "match_confidence": 0.9893333333333333,
  "confidence_score": 0.9,
  "classification_status": "auto_accepted",
  "category_id": "subcat_runtime_issues_connectivity_api_failure",
  "category_name": "Runtime Issues > Connectivity > API Failure: API unreachable or failing"
}
```

LLM fallback (no safe reuse):

```json
{
  "ticket_text_sanitised": "Unable to create articles in a content management system.",
  "confidence_score": 0.9,
  "classification_status": "auto_accepted",
  "category_id": "subcat_design_issues_functional_logic_functional_logic_failure",
  "category_name": "Design Issues > Functional Logic > Functional Logic Failure: Incorrect business logic causes incorrect outcomes"
}
```

Quality gate triggered (review queue outcome):

```json
{
  "ticket_text_sanitised": "In relation to [TICKET_ID], the issue has not been resolved. Please reopen.",
  "confidence_score": 0.3,
  "classification_status": "unclassified",
  "category_id": "UNCLASSIFIED",
  "category_name": "Non-Issue > Non-Issue > No Issue Found"
}
```

---

## What It Doesn't Do

Intellectual honesty about limits is the thing that makes expert readers trust documentation. Here is what this framework does not do:

**No evaluation harness yet.** There is no automated way to measure per-category accuracy. Quality assurance relies on the review queue (0.3% of records flagged for human inspection) and spot-checks. A gold set comparison — comparing predictions against known-correct labels — is on the roadmap but not implemented.

**Sequential processing only.** Records are processed one at a time. There is no parallelism across rows. This is a deliberate trade-off: simplicity of checkpoint correctness and predictable resource usage on a laptop outweigh the complexity of parallel processing.

**Similarity thresholds require dataset-specific calibration.** The 0.975 semantic threshold and 0.96 fuzzy threshold were tuned for incident ticket text. Different datasets — shorter texts, more structured fields, different vocabulary distributions — may need different values. Starting conservative and loosening after sampling is the recommended approach.

---

## The Transferable Insight

The design pattern here is not classifier-specific. Any system that needs to process large volumes of similar inputs with an expensive AI model benefits from the same structure:

1. Define a waterfall of strategies ordered by cost and determinism
2. Apply strict quality guards before any reuse decision
3. Treat the LLM as the last resort, not the first
4. Make every decision auditable: which strategy was used, how confident, how similar the match

The engineering discipline — circuit breakers, checkpoint atomicity, bounded caches, crash-safe deduplication — is what separates a demo from a production system. These are not AI engineering problems. They are software engineering problems that only appear when AI is embedded in a real pipeline.

The "AI skill" in this project was not writing better prompts. It was designing the system around the model, not the model into the system.

---

## Limits / when not to use

- If your taxonomy is unstable or political, the review workload will move from labeling to arguing.
- Similarity reuse (fingerprints, embeddings) is domain-dependent; thresholds that work for one corpus will not generalise unchanged.
- Local inference is a constraint choice, not a free win; throughput depends heavily on hardware and output schema.


## Repro notes

- Run conditions: 2026-04; Apple M3 Pro MacBook, 18GB RAM; local inference (Ollama) `qwen2.5:7b`; embeddings `nomic-embed-text`; bulk output cap `bulk_num_predict: 128` (standard `num_predict: 600`); strict JSON schema enabled
- Dataset class: enterprise incident backlog (63,053 records; anonymised; no client identifiers)
- Non-reproducible from this article: exact prompt schemas, proprietary taxonomies, and internal contracts

---
