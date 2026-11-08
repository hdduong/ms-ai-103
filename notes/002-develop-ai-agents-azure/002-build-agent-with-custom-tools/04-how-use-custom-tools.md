# 04 How to integrate custom tools

[How to integrate custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/4-how-use-custom-tools?pivots=text)

- The model selects a tool using its name, description, and parameter schema. For a local function tool, your application dispatches the returned `function_call` to the allowed Python function and submits its result. Hosted OpenAPI/Azure Function tools have different execution plumbing.
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
    agent_name="snowfall-agent",
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
    agent_name="snowfall-function-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="Use the Azure Function when a city is asked.",
        tools=[tool],
    ),
)
print(agent.name)
```

## OpenAPI tool

The JSON file describes the HTTP API: here, `GET https://wttr.in/{location}?format=j1`. Foundry reads the specification and invokes the API. `security: []` is the standard OpenAPI declaration for no authentication requirement; the tool separately uses `OpenApiAnonymousAuthDetails`. Other supported tool authentication options include an API key or managed identity. A top-level `auth: []` is not an OpenAPI security declaration.

The Learn sample assigns the object to `tool` and then passes `openapi_tool`. That second name is not defined. Pass `tool`.

```json
{
  "openapi": "3.0.0",
  "info": {"title": "get weather data", "version": "v1.0.0"},
  "servers": [{"url": "https://wttr.in"}],
  "security": [],
  "paths": {
    "/{location}": {
      "get": {
        "operationId": "GetCurrentWeather",
        "parameters": [
          {"name": "location", "in": "path", "required": true, "schema": {"type": "string"}},
          {"name": "format", "in": "query", "required": true, "schema": {"type": "string", "default": "j1"}}
        ],
        "responses": {"200": {"description": "Weather response", "content": {"application/json": {"schema": {"type": "object"}}}}}
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
    agent_name="openapi-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="You are a weather assistant. Use the API to fetch weather data.",
        tools=[tool],
    ),
)
print(agent.name)
```

## Exercise: run the function the agent picked

[The complete astronomy exercise](05-exercise-custom-functions.md) includes all three functions, strict schemas, environment setup, the full dispatch loop, verification prompts, and cleanup. Its loop repeatedly handles `function_call` items, dispatches only names in an allowlist, and returns each result under the matching `call_id` on the same conversation.

The snippets above illustrate tool definitions and assume `project_client` already exists. Use `agent_name=`, which is the current `create_version` parameter in the lab's `azure-ai-projects==2.3.0` SDK. Snowfall values in the local example are mock data.
