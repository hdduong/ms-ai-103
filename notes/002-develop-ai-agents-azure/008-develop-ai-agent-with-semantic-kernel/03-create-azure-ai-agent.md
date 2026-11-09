# 03 Create an Azure AI agent with Microsoft Agent Framework

[Create an Azure AI agent with Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/3-create-azure-ai-agent?pivots=text)

- You need the project endpoint and the deployment name. `DefaultAzureCredential` is Azure CLI locally and managed identity in production. No key in code.
- Order: Foundry chat client, then an agent with instructions and optional tools, then `run`.
- One `run` is one user message and one reply, even if tool calls happen inside it. On Foundry the history stays in the service, so a restart or another instance can continue the thread (keep the session id). Other providers keep it in memory and lose it when the process stops.
- Non-streaming waits for the full reply. Streaming yields chunks for a UI. Both expose `text` for the full answer.

## One run

Lab: [Develop an Azure AI chat agent with the Microsoft Agent Framework SDK](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html). The credential in that file is `AzureCliCredential`.

```bash
pip install python-dotenv azure-identity==1.25.3 "pydantic>=2,<3" agent-framework-core==1.12.1 agent-framework-foundry==1.10.3
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from dotenv import load_dotenv

async def main():
    load_dotenv()
    client = FoundryChatClient(
        project_endpoint=os.environ["PROJECT_ENDPOINT"],
        model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
        credential=AzureCliCredential(),
    )
    async with Agent(
        client=client,
        name="ExpenseClaimAgent",
        instructions="Create an expense claim email from the data you are given. Do not ask for more information.",
    ) as agent:
        response = await agent.run("Submit this claim: taxi 42, lunch 18.")
        print(response.text)

asyncio.run(main())
```

The framework also reads `FOUNDRY_PROJECT_ENDPOINT` and `FOUNDRY_MODEL` on its own. This lab passes its own names explicitly.
