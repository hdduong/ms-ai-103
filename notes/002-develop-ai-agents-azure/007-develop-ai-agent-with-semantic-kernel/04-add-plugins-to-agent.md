# 04 Add tools to an Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

- Service tools (Foundry runs them): code interpreter, file search, web search, hosted MCP, Azure AI Search, Foundry toolboxes. Some of these are still preview.
- Custom tools are Python functions you pass in. The framework builds the schema from type hints and the docstring, or from `@tool` (`name`, `description`, optional schema).
- Pass a tool list. The model can choose a tool call and the framework executes local function tools. Tool configuration and approval controls still determine which calls may run.
- `approval_mode` on `@tool` is `always_require` or `never_require`. Use `always_require` before a call that spends money, deletes data, or touches secrets.
- An agent can be exposed as another agent's tool. That is delegation: the caller receives the result and continues. A handoff transfers conversational control and is a different orchestration pattern.
- Describe the function and every parameter. Give each tool a clear job and return a useful result. Decide how to report errors deliberately; do not return a success receipt when an operation failed.

## A function tool

The exercise tool **prints** an email; it does not send one. `approval_mode="never_require"` permits this simulated local tool call without a framework approval pause. Use the exact dependencies and `.env` in [05-exercise.md](05-exercise.md).

```python
import asyncio
import os
from typing import Annotated
from agent_framework import Agent, tool
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from dotenv import load_dotenv
from pydantic import Field

@tool(approval_mode="never_require")
def submit_claim(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")],
):
    """Simulate an expense submission by printing it; send no email."""
    print("To:", to)
    print("Subject:", subject)
    print(body)
    return "Simulated submission printed; no email was sent."

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
        instructions="Create an expense claim addressed to expenses@contoso.com. Subject is Expense Claim. Call the simulated submission tool and clearly report that no email was sent.",
        tools=[submit_claim],
    ) as agent:
        response = await agent.run("Submit this claim: taxi 42, lunch 18.")
        print(response.text)

asyncio.run(main())
```
