# 08 Use workflows in code

[Use workflows in code](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/8-use-workflows-in-code?pivots=text)

- Design the workflow in the portal. Call it by name. You can also download the YAML.
- `input` is the user text, a ticket, a payload, or an empty string that only starts the flow.
- Stream events: `response.completed` is the final text. `response.output_item.done` with a workflow action is one node finishing.
- A human-in-the-loop pause needs another message on the same conversation to continue.
- `type` is `agent_reference`. This unit puts that object under the key `agent`.

## Invoke

Lab: [Build a workflow in Microsoft Foundry](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html). The name is the one you saved in the designer. `input="Start"` only starts the flow. The tickets are already in `Local.SupportTickets`.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

workflow = {"name": "ContosoPay-Customer-Support-Triage"}
with (
    DefaultAzureCredential() as credential,
    AIProjectClient(
        endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
        credential=credential,
    ) as project_client,
    project_client.get_openai_client() as openai_client,
):
    conversation = openai_client.conversations.create()
    stream = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": workflow["name"], "type": "agent_reference"}},
        input="Start",
        stream=True,
    )
    for event in stream:
        if event.type == "response.completed":
            response = openai_client.responses.retrieve(event.response.id)
            print(response.output_text)
    openai_client.conversations.delete(conversation_id=conversation.id)
```
