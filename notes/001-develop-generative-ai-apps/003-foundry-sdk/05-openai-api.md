# 05 Chat Completions API

[Generate responses with the Chat Completions API](https://learn.microsoft.com/en-us/training/modules/foundry-sdk/05-openai-api?pivots=text)

- Use this when you need the older API or wider model compatibility. New Foundry apps should use Responses.
- Same OpenAI-compatible client as unit 04.
- The prompt is a list of `messages`. `system` is the instructions. `user` is the prompt. `assistant` is a previous reply.
- Read `completion.choices[0].message.content`. There is no `output_text` and no `previous_response_id`.
- Next turn: append the assistant reply, append the new user message, and send the whole list again.

The portal has no Chat Completions screen. **Continue in code** generates `responses.create` with `input`, which is unit 04.

## One reply

```python
completion = openai_client.chat.completions.create(
    model="gpt-4.1",  # deployment name
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "When was Microsoft founded?"},
    ],
)
print(completion.choices[0].message.content)
```

## Next turn

```python
conversation_messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "When was Microsoft founded?"},
]
completion = openai_client.chat.completions.create(
    model="gpt-4.1",  # deployment name
    messages=conversation_messages,
)
assistant_message = completion.choices[0].message.content
conversation_messages.append(
    {"role": "assistant", "content": assistant_message},
)
conversation_messages.append(
    {"role": "user", "content": "Who founded it?"},
)
completion = openai_client.chat.completions.create(
    model="gpt-4.1",  # deployment name
    messages=conversation_messages,
)
print(completion.choices[0].message.content)
```
