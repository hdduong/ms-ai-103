# 04 Add tools to an Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

- Service tools (Foundry runs them): code interpreter, file search, web search, hosted MCP, Azure AI Search, Foundry toolboxes. Some of these are still preview.
- Custom tools are Python functions you pass in. The framework builds the schema from type hints and the docstring, or from `@tool` (`name`, `description`, optional schema).
- Pass a list. The model picks the tool and the framework runs it. You do not route.
- `approval_mode` on `@tool` is `always_require` or `never_require`. Use `always_require` before a call that spends money, deletes data, or touches secrets. Your app then has to show the request and resume with the decision.
- One agent can be a tool of another. The caller gets the result and carries on. That is delegation, not the handoff pattern in the multi-agent module.
- Describe the function and every parameter. Return data the model can read. One job per tool. Return an error string instead of raising, so the model can explain the failure.

## Lab: expense claim agent

Lab: [Develop an Azure AI chat agent with the Microsoft Agent Framework SDK](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html). Same packages and `.env` as [03](03-create-azure-ai-agent.md#one-run). `data.txt` from the lab sits next to the script. The tool **prints** the email; nothing is sent.

```python
import asyncio
import os
from pathlib import Path
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
    """Print the claim email."""
    print("To:", to)
    print("Subject:", subject)
    print(body)
    return "Claim email printed. No email was sent."

async def main():
    load_dotenv()
    expenses = Path("data.txt").read_text(encoding="utf-8")
    print(expenses)
    prompt = input("What would you like me to do with these expenses? ")
    client = FoundryChatClient(
        project_endpoint=os.environ["PROJECT_ENDPOINT"],
        model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
        credential=AzureCliCredential(),
    )
    async with Agent(
        client=client,
        name="ExpenseClaimAgent",
        instructions="""You are an AI assistant for expense claim submission.
        At the user's request, create an expense claim and use the plug-in function to send an email to expenses@contoso.com
        with the subject 'Expense Claim' and a body that contains itemized expenses with a total.
        Don't ask for any more information from the user, just use the data provided.""",
        tools=[submit_claim],
    ) as agent:
        response = await agent.run([f"{prompt}: {expenses}"])
        print(response.text)

asyncio.run(main())
```

- Ask `Submit a claim for these expenses.` The lab data is taxi 24.00, dinner 65.50, hotel 125.90, so the total should be **215.40**. Check the model's math.
- The lab's tool returns nothing. Returning a short string tells the model the print worked.
