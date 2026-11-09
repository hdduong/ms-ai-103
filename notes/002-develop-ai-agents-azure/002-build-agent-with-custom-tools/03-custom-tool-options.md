# 03 Options for implementing custom tools

[Options for implementing custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/3-custom-tool-options?pivots=text)

- **Function calling.** You describe the function. The model returns the name and the arguments. Your code runs it. Any language that can host the agent.
- **Azure Functions.** A trigger starts the function. A binding connects the input and the output. The agent sample uses a storage queue in and a storage queue out.
- **OpenAPI 3.0.** The spec is the tool. Foundry calls the HTTP API. Auth is anonymous, API key, or managed identity.
- **Logic Apps.** Low-code workflow that connects apps and data. No SDK sample in this unit.

In the portal, the spec-based tools are under **Tools > Connect a tool > Custom**. A function tool has no portal entry. It lives in your code.

![Select a tool, Custom tab: OpenAPI tool, Model Context Protocol (MCP), and Agent2agent (A2A).](images/03-custom-tools.png)

## Describe the three functions to the model

A function tool separates the **implementation** from the **contract**. The functions in [unit 02](02-why-use-custom-tools.md) read files, calculate, and write a report. `FunctionTool` only tells the model each function's name, purpose, and arguments; registering it does not upload or execute Python.

Save the following as `tools.py` next to `functions.py`. It completes the tool definitions from the [custom-function exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html) using a helper to avoid repeating the schema structure.

```python
from azure.ai.projects.models import FunctionTool


def function_tool(name, description, properties):
    return FunctionTool(
        name=name,
        description=description,
        parameters={
            "type": "object",
            "properties": properties,
            "required": list(properties),
            "additionalProperties": False,
        },
        strict=True,
    )


LOCATION = {"type": "string", "description": "Continent, e.g. south_america."}
COST_PROPERTIES = {
    "telescope_tier": {"type": "string", "enum": ["standard", "advanced", "premium"]},
    "hours": {"type": "number", "description": "Positive number of rental hours."},
    "priority": {"type": "string", "enum": ["low", "normal", "high", "urgent"]},
}
TOOLS = [
    function_tool("next_visible_event", "Find the next event in the sample data.",
                  {"location": LOCATION}),
    function_tool("calculate_observation_cost", "Calculate a telescope rental estimate.",
                  COST_PROPERTIES),
    function_tool("generate_observation_report", "Write a local sample observation report.",
                  {"event_name": {"type": "string"}, "location": LOCATION,
                   **COST_PROPERTIES, "observer_name": {"type": "string"}}),
]
```

- `name` must match a function that the application's dispatch table allows.
- `description` helps the model decide when the tool is relevant.
- `properties` defines argument names and types. Here every property is required, and `additionalProperties=False` rejects extra fields under the strict schema.
- `enum` narrows the tier and priority to the supported values. Python still checks the numeric value of `hours` and performs the calculation.

A request for an event can produce a call named `next_visible_event` with arguments `{"location": "south_america"}`. That call is a request for the application to do work. [Unit 04](04-how-use-custom-tools.md#lab-astronomy-agent) runs it and returns the result with the matching `call_id`.

## Other execution options

The following fragments illustrate the other options in the Learn unit. They assume an initialized `project_client` as shown in [unit 04](04-how-use-custom-tools.md#connect-the-functions-to-the-agent); they are alternatives to the local astronomy functions, not extra files required by that example. Provision the corresponding queue/function or HTTP API before using one.

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

The JSON file is the HTTP API. This one is `GET https://wttr.in/{location}?format=j1`. Foundry reads the spec and sends the request. `security: []` means no auth. The Learn sample writes `auth: []`, which is not an OpenAPI field. Auth on the tool is anonymous, an API key, or a managed identity.

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
