The LLM that builds your system is not the LLM that runs inside it.

In my setup, the same codebase runs one loop in the IDE and another in the pipeline.

One is the build loop: a cloud coding assistant (Claude Code / Trae / Qwen Code) helps me write and refactor in the IDE. It proposes diffs. I approve them. Tests and hooks act as the guardrails.

The other is the run loop: the pipeline executes locally via Ollama (e.g. qwen2.5:7b, qwen3.5:9b, plus a local embedder like nomic-embed-text). This is the part that touches real operational data, so "data stays inside the system" is non-negotiable.

That constraint forces a clean boundary: the cloud assistant never sees raw inputs or outputs.
It sees LLMOps artefacts: a run ID, a manifest, and structured telemetry (hashes and metrics rather than raw content).

That’s enough to iterate safely.
One small example: turning on "thinking mode" looked like it should help, but it could burn the generation budget and come back empty. The fix was boring and explicit: default thinking off for publication runs and cap retries.

If you’re building LLM systems, I think it helps to ask: which role is the model playing, and who catches it first when it’s wrong?

#AI #AIEngineering #SoftwareArchitecture #ClaudeCode #Trae
