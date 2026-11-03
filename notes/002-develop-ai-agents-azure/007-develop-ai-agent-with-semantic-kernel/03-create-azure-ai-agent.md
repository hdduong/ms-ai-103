# 03 Create an Azure AI agent with Microsoft Agent Framework

[Create an Azure AI agent with Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/3-create-azure-ai-agent?pivots=text)

- You need the project endpoint and the deployment name. `DefaultAzureCredential` is Azure CLI locally and managed identity in production. No key in code.
- Order: Foundry chat client, then an agent with instructions and optional tools, then `run`.
- One `run` is one user message and one reply. On Foundry the history stays in the service, so a restart or another instance still has the thread. Other providers keep it in memory and lose it when the process stops.
- Non-streaming waits for the full reply. Streaming yields chunks for a UI. Both expose `text` for the full answer.

## One run

```python
import asyncio
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

async def main():
    agent = Agent(
        client=FoundryChatClient(
            credential=DefaultAzureCredential(),
            project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
            model="gpt-4.1",  # deployment name
        ),
        name="healthcare-assistant",
        instructions="You help staff schedule appointments. Never share medical records.",
    )
    response = await agent.run("What appointment types can I book?")
    print(response.text)

asyncio.run(main())
```
