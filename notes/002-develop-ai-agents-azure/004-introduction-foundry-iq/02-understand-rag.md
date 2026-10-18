# 02 Understanding RAG for agents

[Understanding RAG for agents](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/2-understand-rag?pivots=text)

- A model-only agent misses private data, anything after its cutoff, and your process. It will invent an answer. Every team that builds its own index repeats the same work.
- RAG: **retrieve**, **augment** the prompt, **generate**.
- You get current facts, a source the user can check, and answers tied to your documents.
- Foundry IQ is that retrieval layer as a shared service, so you do not stand up a new index per agent.
