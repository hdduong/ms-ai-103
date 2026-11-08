# 05 Configure retrieval with Foundry IQ

[Configure retrieval with Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/5-configure-retrieval?pivots=text)

- A vague instruction ("use the knowledge base") lets the model answer from training data or skip the citation.
- Say three things: always search first, the citation format, and what to say when the base has no answer.
- Test four prompts: a fact, a question that needs several documents, a question outside the base, and an ambiguous one.
- A good reply is grounded, cited, relevant, and complete.
- Support agents should refuse to guess. Research agents may combine sources and should cite all of them. Compliance agents stay inside the policy text and name the section and effective date.
- In production, watch citation rate, how often it says it does not know, and whether the retrieved docs actually answer the question.
- `type` is `agent_reference`. This unit puts that object under the key `agent`. The call runs the saved agent, so there is no `model` argument here.

## Instructions from the exercise

The lab sets these instructions on the agent, then adds Foundry IQ under Knowledge. The client does not pass `model`. It passes `agent_reference`.

![product-expert-agent instructions: always search the knowledge base for product questions and cite sources.](images/05-agent-instructions.png)

The knowledge base has its own retrieval settings: the chat model that plans queries, **Retrieval reasoning effort**, and **Retrieval instructions** that say which source to try first.

![knowledgebase301 settings: gpt-5 as the planning model, retrieval reasoning effort Minimal, retrieval instructions, and the knowledge sources.](images/05-knowledge-base-config.png)

```python
instructions = """
You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products.
You must ALWAYS search the knowledge base to answer questions about our products or product catalog.
Provide detailed, accurate information and always cite your sources.
If you don't find relevant information in the knowledge base, say so clearly.
"""
print(instructions)
```

## Ask, then approve the knowledge-base call

```python
openai_client.conversations.items.create(
    conversation_id=conversation.id,
    items=[{"type": "message", "role": "user", "content": "What tents does Contoso sell?"}],
)
response = openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    input="",
)
while True:
    approval_requests = [
        item for item in (response.output or [])
        if item.type == "mcp_approval_request"
    ]
    if not approval_requests:
        break
    openai_client.conversations.items.create(
        conversation_id=conversation.id,
        items=[{
            "type": "mcp_approval_response",
            "approval_request_id": item.id,
            "approve": True,
        } for item in approval_requests],
    )
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
print(response.output_text)
```
