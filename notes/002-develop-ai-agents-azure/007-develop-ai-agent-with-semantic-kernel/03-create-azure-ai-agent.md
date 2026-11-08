# 03 Create an Azure AI agent with Microsoft Agent Framework

[Create an Azure AI agent with Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/3-create-azure-ai-agent?pivots=text)

- You need the project endpoint and the deployment name. `DefaultAzureCredential` is Azure CLI locally and managed identity in production. No key in code.
- Order: Foundry chat client, then an agent with instructions and optional tools, then `run`.
- One `run` is one user message and one reply. On Foundry the history stays in the service, so a restart or another instance still has the thread. Other providers keep it in memory and lose it when the process stops.
- Non-streaming waits for the full reply. Streaming yields chunks for a UI. Both expose `text` for the full answer.

## One run

Lab: [Develop an Azure AI chat agent with the Microsoft Agent Framework SDK](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html). The credential in that file is `AzureCliCredential`.

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential

async def main():
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),  # deployment name
        credential=AzureCliCredential(),
    )
    async with Agent(
        client=client,
        name="ExpenseClaimAgent",
        instructions="Create an expense claim email from the data you are given. Do not ask for more information.",
    ) as agent:
        response = await agent.run("Submit this claim: taxi 42, lunch 18.")
        print(response)

asyncio.run(main())
```
