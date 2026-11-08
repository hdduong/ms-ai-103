# 06 Use the function tool

[Use the function tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/06-function?pivots=text)

- The model does not run your code. It returns a `function_call`. Your app runs the function, appends `function_call_output`, and calls `responses.create` again.
- Append `response.output` to the message list before the tool result, and keep the same `call_id`.
- Validate arguments. Log calls. Do not let a tool perform a sensitive action without a check.

```python
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    input=messages,
    tools=[
        {"type": "function", "name": "get_time", "description": "Get the current time"},
    ],
)
messages += response.output
for item in response.output:
    if item.type == "function_call" and item.name == "get_time":
        messages.append({
            "type": "function_call_output",
            "call_id": item.call_id,
            "output": "The time is 2026-03-19 17:17:41.",
        })
        response = openai_client.responses.create(
            model="gpt-4.1",  # deployment name
            instructions="Answer only with the tool output.",
            input=messages,
            tools=[
                {"type": "function", "name": "get_time", "description": "Get the current time"},
            ],
        )
print(response.output_text)
```
