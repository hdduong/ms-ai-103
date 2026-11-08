# 04 How to integrate custom tools

[How to integrate custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/4-how-use-custom-tools?pivots=text)

- The agent is declarative. A clear name and parameter description is what makes it call the tool. You do not write the if-statement that calls it.
- `model` is the deployment name.

## Function tool

JSON schema, `required`, `additionalProperties: False`, `strict=True`. Pass the tool list on `PromptAgentDefinition`.

```python
import json
from azure.ai.projects.models import FunctionTool, PromptAgentDefinition

def recent_snowfall(location: str) -> str:
    """Get recent snowfall totals for a city."""
    totals = {"Seattle": "0 inches", "Denver": "2 inches"}
    snow = totals.get(location, "Data not available.")
    return json.dumps({"location": location, "snowfall": snow})

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

## Azure Function tool

Input queue and output queue, plus the function name and the same kind of parameter schema.

```python
from azure.ai.projects.models import (
    AzureFunctionBinding,
    AzureFunctionDefinition,
    AzureFunctionDefinitionFunction,
    AzureFunctionStorageQueue,
    AzureFunctionTool,
    PromptAgentDefinition,
)

tool = AzureFunctionTool(
    azure_function=AzureFunctionDefinition(
        input_binding=AzureFunctionBinding(
            storage_queue=AzureFunctionStorageQueue(
                queue_name="snowfall-in",
                queue_service_endpoint="https://{storage}.queue.core.windows.net",
            ),
        ),
        output_binding=AzureFunctionBinding(
            storage_queue=AzureFunctionStorageQueue(
                queue_name="snowfall-out",
                queue_service_endpoint="https://{storage}.queue.core.windows.net",
            ),
        ),
        function=AzureFunctionDefinitionFunction(
            name="queue_trigger",
            description="Get snowfall for a city.",
            parameters={
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "City name."},
                },
            },
        ),
    ),
)
agent = project_client.agents.create_version(
    name="snowfall-function-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="Use the Azure Function when a city is asked.",
        tools=[tool],
    ),
)
print(agent.name)
```

## OpenAPI tool

The JSON file is the HTTP API. This one is `GET https://wttr.in/{location}?format=j1`. `auth: []` means no key. Foundry reads the spec and sends the request. Auth on the tool is anonymous, an API key, or a managed identity.

The Learn sample assigns the object to `tool` and then passes `openapi_tool`. That second name is not defined. Pass `tool`.

```json
{
  "openapi": "3.0.0",
  "info": {"title": "get weather data", "version": "v1.0.0"},
  "servers": [{"url": "https://wttr.in"}],
  "paths": {
    "/{location}": {
      "get": {
        "operationId": "GetCurrentWeather",
        "parameters": [
          {"name": "location", "in": "path", "required": true, "schema": {"type": "string"}},
          {"name": "format", "in": "query", "required": true, "schema": {"type": "string", "default": "j1"}}
        ]
      }
    }
  }
}
```

```python
import json
from azure.ai.projects.models import (
    OpenApiAnonymousAuthDetails,
    OpenApiFunctionDefinition,
    OpenApiTool,
    PromptAgentDefinition,
)

with open("weather_openapi.json", "r") as spec_file:
    openapi_weather = json.load(spec_file)

tool = OpenApiTool(
    openapi=OpenApiFunctionDefinition(
        name="get_weather",
        spec=openapi_weather,
        description="Retrieve weather information for a location.",
        auth=OpenApiAnonymousAuthDetails(),
    ),
)
agent = project_client.agents.create_version(
    name="openapi-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="You are a weather assistant. Use the API to fetch weather data.",
        tools=[tool],
    ),
)
print(agent.name)
```

## Exercise: run the function the agent picked

Lab: [Use a custom function in an AI agent](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html). The model returns `function_call`. Your code runs it and sends `function_call_output` on the same conversation.

```python
import json
from openai.types.responses.response_input_param import FunctionCallOutput

conversation = openai_client.conversations.create()
openai_client.conversations.items.create(
    conversation_id=conversation.id,
    items=[{"type": "message", "role": "user", "content": user_input}],
)
input_list = []
response = openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    input=input_list,
)
for item in response.output:
    if item.type == "function_call":
        if item.name == "next_visible_event":
            result = next_visible_event(**json.loads(item.arguments))
        elif item.name == "calculate_observation_cost":
            result = calculate_observation_cost(**json.loads(item.arguments))
        elif item.name == "generate_observation_report":
            result = generate_observation_report(**json.loads(item.arguments))
        input_list.append(
            FunctionCallOutput(
                type="function_call_output",
                call_id=item.call_id,
                output=result,
            )
        )
if input_list:
    response = openai_client.responses.create(
        conversation=conversation.id,
        input=input_list,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    )
print(response.output_text)
```
