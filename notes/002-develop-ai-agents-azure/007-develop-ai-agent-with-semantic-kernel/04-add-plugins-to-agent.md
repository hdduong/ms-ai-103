# 04 Add tools to an Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

- Service tools (Foundry runs them): code interpreter, file search, web search, hosted MCP, Azure AI Search, Foundry toolboxes. Some of these are still preview.
- Custom tools are Python functions you pass in. The framework builds the schema from type hints and the docstring, or from `@tool` (`name`, `description`, optional schema).
- Pass a list. The model picks the tool. You do not route.
- `approval_mode` on `@tool` is `always_require` or `never_require`. Use `always_require` before a call that spends money, deletes data, or touches secrets.
- One agent can be a tool of another. That is the handoff into the multi-agent module.
- Describe the function and every parameter. Return data the model can read. One job per tool. Return an error string instead of raising, so the model can explain the failure.

## A function tool

The exercise tool prints the email. `approval_mode="never_require"` lets the agent send it without a pause.

```python
import asyncio
import os
from typing import Annotated
from agent_framework import Agent, tool
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from pydantic import Field

@tool(approval_mode="never_require")
def submit_claim(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")],
):
    print("To:", to)
    print("Subject:", subject)
    print(body)

async def main():
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),  # deployment name
        credential=AzureCliCredential(),
    )
    async with Agent(
        client=client,
        name="ExpenseClaimAgent",
        instructions="Create an expense claim and use the tool to email expenses@contoso.com. Subject is Expense Claim. Do not ask for more information.",
        tools=[submit_claim],
    ) as agent:
        response = await agent.run("Submit this claim: taxi 42, lunch 18.")
        print(response)

asyncio.run(main())
```
