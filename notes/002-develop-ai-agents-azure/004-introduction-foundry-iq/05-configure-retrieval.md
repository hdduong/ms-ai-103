# 05 Configure retrieval with Foundry IQ

[Configure retrieval with Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/5-configure-retrieval?pivots=text)

- A vague instruction ("use the knowledge base") lets the model answer from training data or skip the citation.
- Say three things: always search first, the citation format, and what to say when the base has no answer.
- Test four prompts: a fact, a question that needs several documents, a question outside the base, and an ambiguous one.
- A good reply is grounded, cited, relevant, and complete.
- Support agents should refuse to guess. Research agents may combine sources and should cite all of them. Compliance agents stay inside the policy text and name the section and effective date.
- In production, watch citation rate, how often it says it does not know, and whether the retrieved docs actually answer the question.
- `type` is `agent_reference`. This unit puts that object under the key `agent`. The call runs the saved agent, so there is no `model` argument here.

## Instructions

```python
from azure.ai.projects.models import PromptAgentDefinition

instructions = """
You are an HR assistant.
Always search the knowledge base before you answer.
Never answer from training data.
Cite each fact as 【doc_id:search_id†source_name】.
If the base has no answer, say you do not have it and tell the user to contact HR.
"""
agent = project_client.agents.create_version(
    name="hr-assistant",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions=instructions,
        tools=[knowledge_tool],
    ),
)
print(agent.name)
```

## Test one question

```python
openai_client = project_client.get_openai_client()
conversation = openai_client.conversations.create()
response = openai_client.responses.create(
    conversation=conversation.id,
    input="How many vacation days do I get?",
    extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
)
print(response.output_text)
```
