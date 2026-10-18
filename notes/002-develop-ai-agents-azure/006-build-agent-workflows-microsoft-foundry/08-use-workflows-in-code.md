# 08 Use workflows in code

[Use workflows in code](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/8-use-workflows-in-code?pivots=text)

- Design in the portal. Call it by name with `type: agent_reference`. You can also download the YAML.
- `input` is the user text, a ticket, a payload, or an empty string that only starts the flow.
- Stream events: `response.completed` is the final text. `response.output_item.done` with a workflow action is one node finishing.
- A human-in-the-loop pause needs another message on the same conversation to continue.

```python
conversation = openai_client.conversations.create()
stream = openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent": {"name": "triage-workflow", "type": "agent_reference"}},
    input="Users can't reset their password from the mobile app.",
    stream=True,
)
for event in stream:
    if event.type == "response.completed":
        for message in event.response.output:
            if message.content:
                for content_item in message.content:
                    if content_item.type == "output_text":
                        print(content_item.text)
```
