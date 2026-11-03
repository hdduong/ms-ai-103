# 04 Add tools to an Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

- Service tools (Foundry runs them): code interpreter, file search, web search, hosted MCP, Azure AI Search, Foundry toolboxes. Some of these are still preview.
- Custom tools are Python functions you pass in. The framework builds the schema from type hints and the docstring, or from `@tool` (`name`, `description`, optional schema).
- Pass a list. The model picks the tool. You do not route.
- `approval_mode` on `@tool` is `always_require` or `never_require`. Use `always_require` before a call that spends money, deletes data, or touches secrets.
- One agent can be a tool of another. That is the handoff into the multi-agent module.
- Describe the function and every parameter. Return data the model can read. One job per tool. Return an error string instead of raising, so the model can explain the failure.

## A function tool

```python
import asyncio
from typing import Annotated
from agent_framework import Agent, tool
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

@tool(approval_mode="never_require")
def recent_snowfall(location: Annotated[str, "City name."]) -> str:
    """Get recent snowfall totals for a city."""
    totals = {"Seattle": "0 inches", "Denver": "2 inches"}
    return totals.get(location, "Data not available.")

async def main():
    agent = Agent(
        client=FoundryChatClient(
            credential=DefaultAzureCredential(),
            project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
            model="gpt-4.1",  # deployment name
        ),
        name="snowfall-agent",
        instructions="Use recent_snowfall when a city is asked.",
        tools=[recent_snowfall],
    )
    response = await agent.run("How much snow fell in Denver?")
    print(response.text)

asyncio.run(main())
```
