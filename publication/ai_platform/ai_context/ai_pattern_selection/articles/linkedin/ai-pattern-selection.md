We almost fine-tuned a model on an internal policy document to "make it know our policies".

The failure mode that would have created: answers that sound authoritative, with no provenance. An expensive way to get it wrong.

Fine-tuning teaches the model how to speak. Retrieval gives it something accurate to say.

Use fine-tuning when behaviour is the problem. Use RAG when knowledge is missing and needs to be cited.

If traceability matters (governance, compliance, architecture decisions), you're in retrieval territory. Training doesn't give you citations.

Considering training because retrieval "isn't good enough"? Fix retrieval first (chunking, metadata, evaluation), then reassess.

Selection guide:

- Need facts from documents that change? Use RAG.
- Need consistent behaviour (tone, format, routing) at scale? Consider fine-tuning or a prompt harness.
- Already have the right material for this task? Use context injection.
- Need deterministic output structure? That's an engineering wrapper. Fine-tuning shapes tendencies; a validator enforces the schema.

Treat fine-tuning like a model release. It can degrade capability outside the target behaviour, so regression-test the tasks you still care about.

#AI #RAG #SoftwareArchitecture #MachineLearning #LLM
