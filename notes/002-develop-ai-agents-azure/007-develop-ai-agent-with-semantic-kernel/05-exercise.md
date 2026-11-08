# 05 Exercise — Build an expense-claim agent

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/5-exercise) · [Official lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html)

Reviewed **2026-11-07** against [Microsoft's instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/07-agent-framework.md) and [starter code](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/agent-framework.py) at commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`. The complete program below is adapted from the exercise. See [MIT attribution](../../../THIRD_PARTY_NOTICES.md).

The Learn URL and this folder retain `semantic-kernel` for link compatibility. The current lesson and exercise use **Microsoft Agent Framework**: `Agent`, `FoundryChatClient`, and `@tool`; they do not use the old `semantic_kernel` SDK.

## What you build

Read expenses from `data.txt`, ask an agent to prepare a claim, and let the agent call a local `submit_claim` function with the recipient, subject, and email body. That function **prints a simulated email**. It does not send mail or submit a real reimbursement.

This demonstrates the full tool cycle: instructions and tool schema reach the model; the model requests a tool call; Agent Framework invokes the local Python function; the result returns to the model for the final response. The tool's parameter annotations explain the inputs to the model.

## 1. Prepare the project and environment

The official lab uses Python **3.13** (tested with **3.13.12**), VS Code with Microsoft's **Foundry Toolkit**, an Azure subscription, and a Foundry project. Deploy the lab's `gpt-5` model where available, or use a deployment supported by the current lab. Copy the project endpoint and **deployment name**; the model catalog name alone is not a resource identifier.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
git -C mslearn-ai-agents checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd mslearn-ai-agents/Labfiles/07-agent-framework/python
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

On PowerShell, use `.\.venv\Scripts\Activate.ps1` to activate. These are the upstream [package requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/requirements.txt):

```text
python-dotenv
azure-identity==1.25.3
pydantic>=2,<3
agent-framework-core==1.12.1
agent-framework-foundry==1.10.3
```

Use a separate virtual environment for each exercise, especially when also trying older Semantic Kernel or Azure Agents SDK examples. These are the lab's direct dependency pins, not a complete transitive lockfile.

Local `.env`:

```dotenv
PROJECT_ENDPOINT=https://YOUR-RESOURCE.services.ai.azure.com/api/projects/YOUR-PROJECT
MODEL_DEPLOYMENT_NAME=YOUR-DEPLOYMENT-NAME
```

The framework also understands native `FOUNDRY_PROJECT_ENDPOINT` and `FOUNDRY_MODEL` settings, but this lab explicitly reads the two names above and passes their values to the client. `AzureCliCredential` uses the identity from `az login`; that identity must have access to this project. Keep `.env` out of commits.

## 2. Complete program

Save this as `expense_claim.py` alongside the official `data.txt`. It combines the lab's imports, tool, client, instructions, data loading, and agent call. The only action tool is the console simulation.

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
    subject: Annotated[str, Field(description="The subject of the email")],
    body: Annotated[str, Field(description="The text body of the email")],
) -> str:
    """Simulate submitting an expense claim by printing an email."""
    print("\n--- Simulated claim email ---")
    print(f"To: {to}\nSubject: {subject}\n\n{body}")
    return "Claim email printed for this exercise. No email was sent."


async def main():
    load_dotenv()
    expenses = Path(__file__).with_name("data.txt").read_text(encoding="utf-8")
    print(f"Expenses:\n{expenses}")
    prompt = input("What would you like me to do with these expenses? ")

    client = FoundryChatClient(
        project_endpoint=os.environ["PROJECT_ENDPOINT"],
        model=os.environ["MODEL_DEPLOYMENT_NAME"],
        credential=AzureCliCredential(),
    )
    async with Agent(
        client=client,
        name="ExpenseClaimAgent",
        instructions=(
            "You are an assistant for expense claim submission. "
            "Create a claim email addressed to expenses@contoso.com with subject "
            "Expense Claim, an itemized list, and the total amount. "
            "Use the submit_claim tool to print the simulated email. "
            "Do not ask for more information. Do not claim that real email was sent."
        ),
        tools=[submit_claim],
    ) as agent:
        response = await agent.run([f"{prompt}:\n{expenses}"])
        print(f"\nAgent response:\n{response.text}")


if __name__ == "__main__":
    asyncio.run(main())
```

The original lab tool returns no value. This adaptation returns a clear simulation receipt so the model can distinguish a printed email from a real send. `os.environ[...]` also fails early when a required setting is missing.

## 3. Run and inspect the tool call

```bash
python expense_claim.py
```

Try: `Submit a claim for these expenses.` The supplied [data file](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/data.txt) contains taxi `24.00`, dinner `65.50`, and hotel `125.90`; their total is **215.40**. Check the printed itemization, total, recipient, and subject. Do not assume the model's arithmetic is correct merely because the tool ran. Real reimbursement code should calculate and validate the amounts deterministically.

`approval_mode="never_require"` is suitable for this print-only demonstration. A tool configured with `always_require` needs an application flow that receives the approval request and resumes the run with the user's decision. Merely changing that string does not build the review interface. Business actions also need application-side validation and authorization; instructions alone are not an enforcement boundary.

## Concepts to retain

| Code | Why it is there |
| --- | --- |
| `load_dotenv()` | Loads the local configuration before constructing the client. |
| `FoundryChatClient(..., model=...)` | Connects Agent Framework to the named model deployment through the Foundry project. |
| `@tool` and `Annotated[..., Field(...)]` | Supply a callable and useful argument descriptions for tool selection. |
| `tools=[submit_claim]` | Makes the local function available to this agent. |
| `await agent.run(...)` | Runs model inference and any resulting function calls; one run can contain several internal model/tool exchanges. |
| `response.text` | Retrieves the final text exposed by the response. |

This is a single-run exercise. Multi-turn use requires reusing an `AgentSession`; resuming after a process restart also requires retaining or restoring the relevant session state or service identifier. Service-side history support does not mean an unrelated new run automatically finds an old conversation.

## Cleanup and troubleshooting

A `KeyError` names a missing environment variable; a model-not-found error commonly means the deployment name does not match the project. Check the signed-in tenant/subscription and project permissions when authentication or authorization fails. For quota failures, follow the current lab's quota guidance or select an eligible deployment in your project.

Closing the Python process ends the local agent session; it does not delete the Azure project or model deployment. Remove dedicated lab resources when finished, preserving shared resources. No actual email was sent by this program.

**Validation:** the example was syntax checked and its APIs compared with the pinned package source. No Azure inference or tool call was executed during this review.
