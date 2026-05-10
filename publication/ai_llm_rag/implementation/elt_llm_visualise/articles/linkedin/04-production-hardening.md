A demo chart pipeline breaks the first time a column name is wrong.
Production needs validation, telemetry, and a failure posture.
So I added hardening gates before any chart renders.

The first production run of my AI visualisation pipeline had five render failures. Every chart the LLM proposed failed Vega-Lite schema validation. Only the index file was written.

The second run of the same config succeeded completely — five charts, six LLM calls, 6 minutes 28 seconds.

📌 Both runs are in the manifest directory. I can diff the configs and trace exactly why one failed and the other succeeded. That is what production governance looks like.

Three mechanisms make the pipeline trustworthy, not just functional:

→ Schema validation at the trust boundary — column references and Vega-Lite specs are validated before any transformation runs or any HTML is written. A hallucinated column name fails loudly at step 1, not silently at render time.

→ Self-critique before rendering — after planning, the LLM reviews its own specs for readability, correctness, and actionability. One additional LLM call. Catches the most common failure: a technically valid chart that is analytically misleading.

→ Run manifests — every run writes a run_manifest.json with config fingerprint, dataset fingerprint, model, LLM call counts, and output file list. Two engineers running the same config against the same data get identical outputs — and the manifest proves it.

The manifest from the successful Seattle weather run: 6 LLM calls, 0 failures, `cost_gbp_est: 0.0` (local inference).

Article linked below — Part 4 of my series on local-first AI visualisation.

What is the failure mode you find hardest to detect in AI-generated outputs?

#DataVisualization #DataEngineering #AI #Python #LLMOps
