# 06 Use the function tool

[Use the function tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/06-function?pivots=text)

- The model returns a `function_call`; your application executes the function and returns `function_call_output`.
- Append `response.output` to history before adding tool results. Match each result to its original `call_id`.
- Handle every function call in a response before requesting the next model response. A response can contain multiple calls.
- Validate names and arguments, and apply the permissions appropriate to each action before executing it.

This example adapts the unit's time function into an executable tool loop. Initialize `openai_client` as shown in [Choose an endpoint and SDK](../003-foundry-sdk/03-choose-an-endpoint-and-sdk.md), and replace the deployment name with your own. It reads the **actual current UTC time**, rather than sending a fixed demonstration timestamp.

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
model_deployment = "gpt-4.1"  # Replace with your tool-capable deployment name.

for _ in range(5):
    response = openai_client.responses.create(
        model=model_deployment,
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
        if json.loads(call.arguments or "{}") != {}:
            raise ValueError("get_time accepts no arguments")
        messages.append({
            "type": "function_call_output",
            "call_id": call.call_id,
            "output": get_time(),
        })
else:
    raise RuntimeError("The model exceeded this example's tool-call limit")
```

The function schema tells the model what it may request; it is not the Python implementation. The app supplies the value and retains all response items, including reasoning items when present. The loop limit prevents an endless sequence of tool requests in this example.

The [official module exercise](07-exercise-use-tools.md) implements web/file search. This custom-function example comes from the separate Learn unit, reviewed **2026-11-07**.
