# 04 How to integrate custom tools

[How to integrate custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/4-how-use-custom-tools?pivots=text)

- The agent is declarative. A clear name and parameter description is what makes it call the tool. You do not write the if-statement that calls it.
- Function tool: JSON schema, `required`, `additionalProperties: False`, `strict=True`. Pass the tool list on `PromptAgentDefinition`.
- Azure Function tool: input and output storage queues, plus the function name and the same parameter schema.
- OpenAPI tool: a spec file, `OpenApiAnonymousAuthDetails` (or key or managed identity), then the same `tools=` list.
- `model` is the deployment name.

```python
function_tool = FunctionTool(
    name="recent_snowfall",
    parameters={
        "type": "object",
        "properties": {
            "location": {"type": "string", "description": "City name."},
        },
        "required": ["location"],
        "additionalProperties": False,
    },
    description="Get recent snowfall totals for a city.",
    strict=True,
)
agent = project_client.agents.create_version(
    name="snowfall-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="You track snowfall. Use the function when a city is asked.",
        tools=[function_tool],
    ),
)
print(agent.name)
```
