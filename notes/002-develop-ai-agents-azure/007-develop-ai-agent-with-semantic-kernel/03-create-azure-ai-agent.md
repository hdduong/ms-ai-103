# 03 Create an Azure AI agent with Microsoft Agent Framework

[Create an Azure AI agent with Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/3-create-azure-ai-agent?pivots=text)

- You need the project endpoint and deployment name. `DefaultAzureCredential` tries a credential chain; it does not always select Azure CLI locally or managed identity in production. This exercise explicitly uses `AzureCliCredential` after `az login`.
- Order: Foundry chat client, then an agent with instructions and optional tools, then `run`.
- A `run` can include multiple model turns and tool calls before returning an `AgentResponse`. Reuse an `AgentSession` for a multi-turn conversation. To resume after a restart, save and restore the appropriate session state/service conversation identifier; a newly constructed client does not automatically find the old conversation.
- Non-streaming returns a completed response with `.text`. Streaming yields incremental updates for a UI; aggregate those updates when the application needs a final response.

## One run

Lab: [Develop an Azure AI chat agent with the Microsoft Agent Framework SDK](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html). Use the packages and `.env` from [05-exercise.md](05-exercise.md). This shorter example only drafts text; the next unit adds the simulated submission tool.

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
