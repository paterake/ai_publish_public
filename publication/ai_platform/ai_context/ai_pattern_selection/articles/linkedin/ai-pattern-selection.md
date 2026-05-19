We almost fine-tuned a model on an internal policy document to “make it know our policies”.

That would have been the most expensive way to get the wrong failure mode: answers that sound authoritative, but have no provenance.

Here’s the distinction that prevents a lot of wasted effort:

Fine-tuning trains the voice. Retrieval retrieves the facts.

Use fine-tuning when the thing that is broken is behaviour. Use RAG when the thing that is missing is knowledge you need to cite with provenance.

If your use case needs traceability (governance, compliance, architecture decisions), you are already in retrieval territory. Training does not give you citations.

If you’re considering training because retrieval “isn’t good enough”, fix retrieval first (chunking, metadata, evaluation), then reassess.

The practical selection rule:

- Need facts from documents that change? Use RAG.
- Need consistent behaviour (tone, format, routing) at production scale? Consider fine-tuning (or a prompt harness).
- Already have the right material for this specific task? Use context injection and be deliberate about scope.

One more: if you need mandatory structure and completeness, that’s an engineering constraint problem. Fine-tuning can shape tendencies, but it cannot enforce a schema without a wrapper that validates sections and retries thin output.

Also: treat fine-tuning like a model release. It can degrade capability outside the target behaviour, so regression-test the tasks you still care about.

#AI #RAG #SoftwareArchitecture #MachineLearning #LLM
