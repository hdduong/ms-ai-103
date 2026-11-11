# 02 Understanding RAG for agents

[Understanding RAG for agents](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/2-understand-rag?pivots=text)

- A model cannot retrieve your private documents or recent updates from its trained weights. You can supply facts in a prompt; RAG automates finding the relevant facts for each question. Without relevant context, the model may invent an answer.
- RAG is three steps: **retrieve** matching chunks, **augment** the prompt with those chunks, **generate** the reply from that context.
- RAG can use updated documents without a new training run and return references for verification. Check that retrieval found the right content and that the reply actually follows it; retrieval alone does not guarantee correctness.
- Foundry IQ is that stack as a shared service. The next unit is the product.

In the exercise, `product-expert-agent` gets its retrieve step from `knowledgebase301` under **Knowledge**.

![product-expert-agent in the portal. Knowledge lists knowledgebase301. Web search is the only other tool.](images/02-agent-knowledge.png)

The lab's portal setup is in [04](04-data-requirements.md#lab-blob-source-for-product-expert-agent). Its Python client, which calls the agent and answers the knowledge-base approval requests, is in [05](05-configure-retrieval.md#lab-ask-then-approve-the-knowledge-base-call).

## Follow a product question through RAG

For `Which backpacks are available in XL?`, **retrieve** means searching the product PDFs, **augment** means returning the relevant catalog passages to the agent, and **generate** means writing an answer that cites those passages. The Python client in unit 05 requests an agent response; it does not implement an embedding pipeline or retrieve arbitrary files from Blob itself. Foundry IQ handles retrieval through the connected knowledge-base tool.
