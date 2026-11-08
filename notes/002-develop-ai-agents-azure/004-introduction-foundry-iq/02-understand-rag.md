# 02 Understanding RAG for agents

[Understanding RAG for agents](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/2-understand-rag?pivots=text)

- A model with no retrieval misses new facts, private docs, your process, and it will invent an answer. Every team then rebuilds the same search stack.
- RAG is three steps: **retrieve** matching chunks, **augment** the prompt with those chunks, **generate** the reply from that context.
- What you get: answers stay current without a new training run, the reply can name the source, and the answer is tied to your documents.
- Foundry IQ is that stack as a shared service. The next unit is the product.

In the exercise, `product-expert-agent` gets its retrieve step from `knowledgebase301` under **Knowledge**.

![product-expert-agent in the portal. Knowledge lists knowledgebase301. Web search is the only other tool.](images/02-agent-knowledge.png)

The exercise client is the retrieve step. It sends the question to the agent that has the knowledge base, then answers `mcp_approval_request` until the reply has no pending tool call. Lab: [Integrate an AI agent with Foundry IQ](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04-integrate-agent-with-foundry-iq.html).

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential(
    exclude_environment_credential=True,
    exclude_managed_identity_credential=True,
)
project_client = AIProjectClient(
    credential=credential,
    endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
)
openai_client = project_client.get_openai_client()
agent = project_client.agents.get(agent_name="product-expert-agent")
conversation = openai_client.conversations.create(items=[])

user_message = "What tents does Contoso sell?"
openai_client.conversations.items.create(
    conversation_id=conversation.id,
    items=[{"type": "message", "role": "user", "content": user_message}],
)
response = openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    input="",
)
while True:
    approval_requests = [
        item for item in (getattr(response, "output", None) or [])
        if getattr(item, "type", None) == "mcp_approval_request"
    ]
    if not approval_requests:
        break
    approval_items = []
    for approval_request in approval_requests:
        approval_items.append({
            "type": "mcp_approval_response",
            "approval_request_id": approval_request.id,
            "approve": True,
        })
    openai_client.conversations.items.create(
        conversation_id=conversation.id,
        items=approval_items,
    )
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
print(response.output_text)
```
