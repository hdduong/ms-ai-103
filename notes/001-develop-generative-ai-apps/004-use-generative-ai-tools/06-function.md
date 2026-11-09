# 06 Use the function tool

[Use the function tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/06-function?pivots=text)

- The model does not run your code. It returns a `function_call`. Your app runs the function, appends `function_call_output`, and calls `responses.create` again.
- Append `response.output` to the message list before the tool result, and keep the same `call_id`.
- One response can hold several calls. Answer all of them before the next request.
- Validate arguments. Log calls. Do not let a tool perform a sensitive action without a check.

The unit's time function as a loop. `openai_client` is from [Choose an endpoint and SDK](../003-foundry-sdk/03-choose-an-endpoint-and-sdk.md).

```python
import json
from datetime import datetime, timezone

def get_time():
    return "The current UTC time is " + datetime.now(timezone.utc).isoformat()

tools = [{
    "type": "function",
    "name": "get_time",
    "description": "Get the current UTC time from the application clock.",
    "parameters": {
        "type": "object",
        "properties": {},
        "required": [],
        "additionalProperties": False,
    },
    "strict": True,
}]
messages = [{"role": "user", "content": "What is the current UTC time?"}]

for _ in range(5):
    response = openai_client.responses.create(
        model="gpt-4.1",  # deployment name
        instructions="Use get_time for the current time and answer using its output.",
        input=messages,
        tools=tools,
    )
    messages.extend(response.output)
    calls = [item for item in response.output if item.type == "function_call"]
    if not calls:
        print(response.output_text)
        break
    for call in calls:
        if call.name != "get_time":
            raise ValueError("Unknown function: " + call.name)
        messages.append({
            "type": "function_call_output",
            "call_id": call.call_id,
            "output": get_time(),
        })
```

The schema tells the model what it can ask for. Your code does the work. `range(5)` stops a runaway loop.
