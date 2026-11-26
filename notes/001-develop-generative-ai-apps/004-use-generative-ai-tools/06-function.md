# 06 Use the function tool

[Use the function tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/06-function?pivots=text)

- The model does not run your code. It returns a `function_call`. Your app runs the function, appends `function_call_output`, and calls `responses.create` again.
- Append `response.output` to the message list before the tool result, and keep the same `call_id`.
- One response can hold several calls. Answer all of them before the next request.
- Validate arguments. Log calls. Do not let a tool perform a sensitive action without a check.

The unit's time function as a loop. `openai_client` is from [Choose an endpoint and SDK](../003-foundry-sdk/03-choose-an-endpoint-and-sdk.md).

```python
from datetime import datetime, timezone

# Runs in your app, not on the service; the model only requests it
def get_time():
    return "The current UTC time is " + datetime.now(timezone.utc).isoformat()

# Describes the call the model can request; it takes no arguments
tools = [{
    "type": "function",
    "name": "get_time",
    "description": "Get the current UTC time from the application clock.",
    "parameters": {
        "type": "object",
        "properties": {},
        "required": [],
        "additionalProperties": False,  # strict mode requires this
    },
    "strict": True,  # enforce the schema on the model's arguments
}]
# The conversation, kept by the app and sent in full each request
messages = [{"role": "user", "content": "What is the current UTC time?"}]

tool_rounds = 0
# Repeat until the model answers instead of asking for a tool
while True:
    # Send the history; the reply is more calls or the final answer
    response = openai_client.responses.create(
        model="gpt-4.1",  # deployment name
        instructions="Use get_time for the current time and answer using its output.",
        input=messages,
        tools=tools,
    )
    # Keep the function_call items; each result must follow its call
    messages.extend(response.output)
    # Calls the model wants this app to run
    calls = [item for item in response.output if item.type == "function_call"]
    # No calls: this response is the answer
    if not calls:
        print(response.output_text)
        break
    # After five rounds, more calls raise rather than stop with no answer
    if tool_rounds >= 5:
        raise RuntimeError("The model requested more than five rounds of tool calls.")
    tool_rounds += 1
    # Answer every call in this response before the next request
    for call in calls:
        # Run only functions you defined; reject anything else
        if call.name != "get_time":
            raise ValueError("Unknown function: " + call.name)
        # Add the result; the matching call_id ties it to the request
        messages.append({
            "type": "function_call_output",
            "call_id": call.call_id,
            "output": get_time(),
        })
```

Unlike the hosted web/file tools in the [travel application](05-file-search.md#lab-brochures-plus-web-search), a custom function runs in your application process.
