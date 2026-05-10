# The LLMOps Foundation: 13 Primitives, Zero Cloud Dependencies

A quality regression appeared in the retrieval pipeline. Without shared infrastructure it
would have been invisible — a worse answer, silently delivered. With it, the eval gate
flagged it in CI, the baseline comparison showed exactly which metric dropped, and the
telemetry stream pointed to which run introduced the change. That is the difference between
a platform you can operate and one you can only hope works.

This article is about the shared substrate that makes multi-module AI operable: 13 LLMOps
primitives, all offline-capable, all OSS-backed, built once and inherited by every module
in the platform.

## Key takeaways

- 13 LLMOps primitives — eval, caching, guardrails, circuit breaking, telemetry, lineage —
  built once, used by ten modules, no cloud dependency required.
- We built custom implementations first (to understand the contracts), then replaced each
  infrastructure primitive with the industry-standard OSS library.
- What remains custom is the integration layer that no single library provides: run
  correlation, content-safe telemetry, manifests, and policy packs.

---

## Who this is for

You are building a multi-module AI system. You have noticed that quality regressions are
invisible, failures are undiagnosable, and every module has reinvented telemetry and retry
logic in its own incompatible way. Or you skipped LLMOps entirely and are now wondering
why production behaviour is unpredictable.

## What you will learn

- What a complete, offline-capable LLMOps substrate looks like — all 13 primitives
- Why we built custom implementations first, and then replaced them with OSS libraries
- What stays custom, and why — the integration layer that no library provides

## Constraints

- Local-first operation, zero data egress — cloud LLM APIs and SaaS LLMOps platforms are
  not available
- Sensitive data cannot appear in logs or telemetry streams

---

## The Problem: What Goes Wrong Without a Shared Substrate

Once a system has five modules — ingestion, query, retrieval, graph, agent — the naive
approach is to let each one define its own config schema, log in its own format, invent its
own run identifiers, and handle retries however it chooses. Everything works. Until it does
not.

A quality regression in the retrieval output has no visible cause. An embedding call starts
failing intermittently — which module? Which run? A batch pipeline silently exhausts its
token budget and returns empty responses. A circuit opens and nobody notices until a user
reports a timeout.

These are not model problems. They are missing infrastructure: no run correlation across
modules, no eval gate, no circuit breaker, no feedback loop. Every module re-implements these
primitives badly and inconsistently.

The turning point is recognising that operational consistency is a feature, not a refactor.
And the right time to build it is before the second module, not after the fifth.

---

## The Naive Approach: Build It All Custom

The initial approach was direct: build the primitives as custom code. Eval harness: a
hand-written judge with expected-based scoring. Circuit breaker: a custom open/half-open/
closed state machine. Retry logic: a manual backoff loop with exception matching. Guardrails:
regex tripwires for PII and injection patterns.

It worked. Each primitive did what it needed to do. But the maintenance surface grew with
every module that consumed them, and a simple question had no good answer: "why didn't you
just use RAGAS?"

The honest answer: because we built the eval harness before we understood what contract it
needed to satisfy. Once the contract was clear — what event shape it must emit, where the
CI boundary must fall, what baseline comparison it must support — the OSS migration became
a mechanical swap, not a redesign.

This is the build-order lesson embedded in the journey: you cannot choose the right library
for a problem you do not yet understand. Build custom first to establish the contract. Then
replace the implementation while keeping the contract intact.

---

## The 13 Primitives

A shared core package (called `elt_llm_core` in this codebase) provides 13 primitives. Every module in the platform can opt in by
calling these helpers — nothing runs unless a module invokes them.

| # | Primitive | What it does |
|---|-----------|-------------|
| 1 | Run correlation (`run_id`) | Propagates a shared identifier across all modules so any telemetry, manifest, or artefact written during a run can be correlated end-to-end |
| 2 | Structured telemetry | Emits JSON events (LLM call, tool call, eval result, feedback) to application logs and optionally to a JSONL file; content-safe by default |
| 3 | Embedding observability | Wraps embedding calls to emit `tool_call` telemetry and track per-run counters (calls, batches, texts, failures) |
| 4 | Lineage manifests | Writes `run_manifest.json` and `corpus_manifest.json` — replayable lineage records keyed by `run_id` |
| 5 | Cost estimation | Token estimation and GBP pricing model; `llm_call` events carry `prompt_tokens_est`, `output_tokens_est`, `cost_gbp_est` |
| 6 | Cost guardrails | Optional per-run budgets enforced before each LLM call; failures emit `error_type=budget_exceeded` telemetry |
| 7 | LlamaIndex + OpenTelemetry tracing | LlamaIndex callback instrumentation with optional OTLP span export; JSONL spans for offline trace analysis |
| 8 | Contract helpers | `exception_to_error` and `normalise_tool_result` — stable error/result envelopes at module boundaries |
| 9 | Evaluation harness | Two-mode dispatch: deterministic expected-based (CI, zero LLM calls) and RAGAS judge-mode (faithfulness, answer relevance, context precision); objective retrieval metrics (Hit@k, MRR, NDCG) |
| 10 | Feedback events (HITL) | Standard feedback schema tied to `run_id`; dataset export utility for privacy-safe analysis |
| 11 | Semantic cache + guardrails + artifact registry | Embedding-based semantic cache (JSONL or Redis backend); semantic guardrails (PII + injection); artifact snapshot/promotion/rollback pointers |
| 12 | Ops reporting + retention | Offline drift/performance report from telemetry; age/size-based cleanup for telemetry, traces, manifests |
| 13 | Artifact promotion / rollback | CLI for promoting corpus, graph, and index snapshots to named pointers (`production.txt`); rollback is a re-promotion |

Every module that consumes the core package gets all 13 primitives. They do not each define
their own config schema, model factory, or telemetry sink. They call the shared helpers and
focus on domain logic.

---

## The OSS Migration

Once the contracts were stable, three custom implementations were replaced with
industry-standard libraries.

### RAGAS — Evaluation

The custom eval harness scored generation quality with hand-written expected comparisons.
RAGAS replaced the judge scoring while preserving the contract: the `eval_result` telemetry
event shape stayed unchanged, the CI boundary (deterministic, zero judge calls) stayed
intact, and the baseline comparison mechanism stayed in place.

The gain: Faithfulness, AnswerRelevancy, and ContextPrecisionWithoutReference are now
industry-standard, citable metrics. Running against a local Ollama endpoint means evaluation
costs £0 in API calls.

```python
# Judge mode now delegates scoring to RAGAS
from ragas.metrics.collections import (
    AnswerRelevancy,
    ContextPrecisionWithoutReference,
    Faithfulness,
)
from ragas import SingleTurnSample

sample = SingleTurnSample(
    user_input=question,
    response=answer,
    retrieved_contexts=contexts,
)
score = await metric.single_turn_ascore(sample)
```

### pybreaker — Circuit Breaker

The custom circuit breaker implemented the open/half-open/closed state machine directly.
`pybreaker` replaced the state machine; the wrapper preserved the telemetry contract —
breaker state transitions are still observable via the standard event stream.

### tenacity — Retry Logic

The manual backoff loop across 67 lines of custom retry handling became this:

```python
from tenacity import (
    AsyncRetrying, retry_if_exception,
    stop_after_attempt, wait_exponential,
)

async for attempt in AsyncRetrying(
    stop=stop_after_attempt(max_retries + 1),
    wait=wait_exponential(multiplier=backoff_base, max=backoff_max),
    retry=retry_if_exception(_retryable_exc),
    before_sleep=_before_sleep,
    reraise=True,
):
    with attempt:
        return await _attempt_once(attempt.retry_state.attempt_number)
```

The retry predicate (`_retryable_exc`) remains explicit — which exceptions are retryable is a
domain decision, not a library default. Observability via `before_sleep` is preserved.

### What was not migrated (and why)

**LLM Guard** for guardrails: blocked. The upstream package declares
`requires-python <3.13`; this platform runs Python 3.13 in CI. The blocker is documented
and monitored. The existing semantic guardrail tripwires remain in place.

**NVIDIA NeMo Guardrails**: evaluated and rejected. NeMo Guardrails is a conversation flow
controller — it handles topic adherence and dialogue routing via a custom DSL (Colang). The
platform's guardrail requirement is input/output scanning for PII and injection. These are
different problems. NeMo would have solved the wrong one while introducing a new maintenance
language.

---

## What Stays Custom: The Integration Layer

After the OSS migrations, the remaining custom code is the integration layer — the glue that
makes disparate OSS tools work together in a coherent, observable, reproducible pipeline.

This is not boilerplate. It is the original contribution:

- **JSONL telemetry + content-safe hashing + `run_id` correlation** — the contract surface
  that makes cross-module debugging possible. Prompt text never appears in logs. Hashes and
  character counts are sufficient to detect regressions without leaking document content.
- **Manifests + fingerprinting** — portable, replayable lineage records. Given a `run_id`
  and a manifest, you can reconstruct exactly what ran, against what data, with what
  configuration.
- **Policy packs** — domain-specific deny/redact patterns for guardrails. These are thin
  YAML config; they will survive the LLM Guard migration because they express project
  requirements, not library defaults.
- **Retrieval metrics (Hit@k, MRR, NDCG)** — deterministic, zero-dependency, no judge model
  required. Candidates for RAGAS migration when retrieval metric support matures.
- **Retention and drift reporting** — specific to the JSONL output stream and directory
  layout. No general-purpose equivalent exists offline.

A reader of the publications cannot find this integration layer in any single library. It is
the assembly — and assembly is where the architectural decisions live.

---

## The CI Gate: Deterministic Evaluation, Zero LLM Calls

The eval harness supports two dispatch modes:

```
CI mode  →  expected-based comparison  →  zero judge/LLM calls
Local    →  RAGAS judge scoring        →  Ollama localhost, £0 API cost
```

The CI boundary is enforced in code: if generation metrics are requested in CI mode without
an `expected` value, the harness raises a `ValueError` before any LLM call is attempted.

```python
# CI gate: generation metrics require expected; never call a judge in CI
def test_ci_mode_requires_expected_for_generation_metrics():
    case = EvalCase(case_id="c1", question="q", answer="a",
                    contexts=["ctx"], expected=None)
    with pytest.raises(ValueError):
        run_eval_case(eval_id="e1", case=case, metrics=["faithfulness"],
                      judge=None, require_expected=True)
```

This test does not depend on an LLM. It verifies that the CI boundary is enforced by the
harness itself, not by a CI environment variable or an external flag. 114 tests pass in
this configuration — zero judge calls, deterministic results.

Objective retrieval metrics (Hit@k, MRR, NDCG) run in the same CI mode. A case provides
`retrieved_ids` and `relevant_ids`; the metric calculation is pure maths.

---

## Content-Safe Telemetry: Observability Without Data Leakage

The telemetry design deserves its own callout for sensitive-data environments.

Every LLM call event records `prompt_chars`, `prompt_hash`, `output_chars`, `output_hash`
— not the raw prompt or output text. Every embedding event does the same for its input.

A representative `llm_call` event:

```json
{
  "event_type": "llm_call",
  "run_id": "3f8a...",
  "module": "retriever",
  "prompt_chars": 4823,
  "prompt_hash": "a7c3...",
  "output_chars": 342,
  "output_hash": "b12f...",
  "prompt_tokens_est": 1205,
  "output_tokens_est": 85,
  "cost_gbp_est": 0.0,
  "duration_ms": 4821
}
```

`cost_gbp_est` is 0.0 for local inference — there is no billing. The field exists so the
same telemetry schema works when a module moves to a cloud provider: switching the pricing
config produces real cost estimates without changing the event shape.

[NEEDS_DATA: replace the example above with a sanitised `llm_call` event from a real run
using `ELT_LLM_TELEMETRY_SINK=jsonl` — the schema shape is correct; the values should come
from an actual run to confirm the field names and formatting in the published article.]

---

## The Lesson: The Decision Gate

Every OSS migration in this project was preceded by the same question: "Is there a library
that solves this?"

For a retry loop, the answer is yes — tenacity. For a circuit breaker state machine, yes
— pybreaker. For RAG evaluation metrics, yes — RAGAS. For PII/injection scanning, yes — LLM
Guard (blocked on Python 3.13, but the choice is made and the migration is queued).

For the integration layer — run correlation, content-safe telemetry schema, lineage
manifests, policy packs — the answer is no. Those are project-specific contracts. They
remain custom because there is a reason for them to be custom.

The decision gate:

> Every line of custom code is a decision to diverge from what already exists. That decision
> needs a reason. "It's faster to write it myself" is not a reason — it is the bias this
> principle exists to correct.

The practical effect of applying this gate: the publications become about the integration
decisions, not the commodity code. Using RAGAS, pybreaker, and tenacity signals to readers
that the author surveyed the field and chose deliberately. Building a retry loop from scratch
signals the opposite.

---

## What Every Module Inherits

In this platform, the ingest/query/retrieval/graph/agent modules all consume the same core package. None of them define their own telemetry sink, model factory, or config schema. They inherit the 13 primitives and focus on domain logic.

The result: a platform where every module is operable on the same terms. Run correlation
works across all of them. Telemetry events share the same schema. Manifests are written to
the same output root. Cost guardrails apply uniformly. When a regression appears, the chain
is traceable end-to-end regardless of which module introduced it.

---

## Limits / When Not to Use

- Telemetry records hashes and character counts, not raw content. If you need input/output
  text for debugging, that is a separate trace mechanism outside this layer.
- Token and cost estimates are derived from character counts — reliable proxies, not
  billing-accurate figures.
- Budget guardrails block at the call level — they do not provide backpressure or queuing.
  A pipeline that hits a budget cap stops, not slows.
- LLM Guard migration is blocked on Python 3.13 upstream compatibility. The existing
  guardrail tripwires handle common patterns; NLP-based PII scanning is pending.
- The retrieval gold set is currently a smoke test placeholder — meaningful CI gating on
  retrieval quality requires domain questions with verified relevant IDs from the live index.

---

## Repro Notes

- Run conditions: 2026-04; Apple M3 Pro, 18GB RAM; `ELT_LLM_TELEMETRY_SINK=jsonl` enabled;
  all inference via Ollama `localhost:11434`; Python 3.13 with UV workspace
- Dataset class: enterprise governance corpus (internal documents; not reproducible from
  this article)
- Test count: 114 tests passing at time of writing; deterministic CI gate with zero LLM
  calls
- RAGAS version: current stable; judge metrics point at Ollama-hosted model via the
  OpenAI-compatible `/v1` endpoint — no cloud API calls
- What is not reproducible from this article: domain-specific config values, prompt schemas,
  internal workload profiles
