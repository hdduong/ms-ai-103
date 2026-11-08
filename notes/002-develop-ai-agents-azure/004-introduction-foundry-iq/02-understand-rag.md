# 02 Understanding RAG for agents

[Understanding RAG for agents](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/2-understand-rag?pivots=text)

- A model with no retrieval misses new facts, private docs, your process, and may invent an unsupported answer. Every team then rebuilds the same search stack.
- RAG is three steps: **retrieve** matching chunks, **augment** the prompt with those chunks, **generate** the reply from that context.
- Retrieval can supply newer information without retraining and support source citations. Freshness depends on source updates/ingestion; grounding and citation correctness still need evaluation.
- Foundry IQ is that stack as a shared service. The next unit is the product.

In the exercise, `product-expert-agent` gets its retrieve step from `knowledgebase301` under **Knowledge**.

![product-expert-agent in the portal. Knowledge lists knowledgebase301. Web search is the only other tool.](images/02-agent-knowledge.png)

The exercise client invokes the saved agent; the agent's knowledge-base tool performs retrieval. The client supplies approval or denial when a response contains `mcp_approval_request`. [The complete Foundry IQ exercise](06-exercise-foundry-iq.md) includes resource setup, the actual knowledge-tool approval configuration, a runnable Python client, test prompts, and cleanup.

An approval policy must be configured on the `kb-knowledgebase...` tool. A loop that always sends `approve=True` is an automatic policy, not a human approval decision.
