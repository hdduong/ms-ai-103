# 04 How to integrate custom tools

[How to integrate custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/4-how-use-custom-tools?pivots=text)

- The model picks the tool from its name, description, and parameters. You do not write the logic that decides. For a function tool, your code still runs the function and sends back `function_call_output`. Azure Function and OpenAPI tools run in Azure.
- `model` is the deployment name.
- `create_version` takes `agent_name=`. The Learn function sample passes `name=`, which fails.

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

## Lab: astronomy agent

Lab: [Use a custom function in an AI agent](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html). The starter `functions.py` has `calculate_observation_cost` and `generate_observation_report`. You write `next_visible_event` ([02](02-why-use-custom-tools.md)).

```bash
pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

`agent.py`. The model returns `function_call`. The app runs the function and sends `function_call_output` on the same conversation until no calls are left.

```python
import json
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import FunctionTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from functions import (
    next_visible_event, calculate_observation_cost, generate_observation_report,
)

FUNCTIONS = {
    "next_visible_event": next_visible_event,
    "calculate_observation_cost": calculate_observation_cost,
    "generate_observation_report": generate_observation_report,
}

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

location = {"type": "string", "description": "continent, e.g. 'south_america'"}
cost = {
    "telescope_tier": {"type": "string", "enum": ["standard", "advanced", "premium"]},
    "hours": {"type": "number"},
    "priority": {"type": "string", "enum": ["low", "normal", "high", "urgent"]},
}
tools = [
    function_tool("next_visible_event", "Find the next visible astronomical event.", {"location": location}),
    function_tool("calculate_observation_cost", "Calculate the cost of telescope time.", cost),
    function_tool(
        "generate_observation_report",
        "Generate a report summarizing an astronomical observation.",
        {"event_name": {"type": "string"}, "location": location, **cost, "observer_name": {"type": "string"}},
    ),
]

load_dotenv()
project_client = AIProjectClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()
agent = project_client.agents.create_version(
    agent_name="astronomy-agent",
    definition=PromptAgentDefinition(
        model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
        instructions="You are an astronomy observations assistant that helps users find information about astronomical events and calculate telescope rental costs.",
        tools=tools,
    ),
)
reference = {"agent_reference": {"name": agent.name, "version": agent.version, "type": "agent_reference"}}
conversation = openai_client.conversations.create()

while True:
    prompt = input("You (exit to stop): ").strip()
    if prompt.lower() in {"exit", "quit"}:
        break
    response = openai_client.responses.create(
        conversation=conversation.id,
        input=prompt,
        extra_body=reference,
    )
    for _ in range(8):
        calls = [item for item in response.output if item.type == "function_call"]
        if not calls:
            break
        outputs = []
        for call in calls:
            function = FUNCTIONS.get(call.name)
            result = function(**json.loads(call.arguments)) if function else json.dumps({"error": "unknown function"})
            outputs.append({"type": "function_call_output", "call_id": call.call_id, "output": result})
        response = openai_client.responses.create(
            conversation=conversation.id,
            input=outputs,
            extra_body=reference,
        )
    print("Agent:", response.output_text)

openai_client.conversations.delete(conversation_id=conversation.id)
project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
```

Try:

```text
Find me the next event I can see from South America and give me the cost for 5 hours of premium telescope time at normal priority.
Generate that information in a report for Bellows College.
```

- The sample rates are premium $300/hour and normal x1.25, so 5 hours is **$1,875**. High priority (x1.75) would be $2,625.
- The report lands in a new `report_<event>_<timestamp>.txt` file next to the script.
- Keep every turn and every result on the one conversation. Mixing in `previous_response_id` can drop a function output.
- Events are sample data with month and day only. Late in the year the search can come back empty.
