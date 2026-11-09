# 04 Add tools to an Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

A tool connects a model's decision to application behavior. Foundry executes hosted tools such as file search or code interpreter; your Python process executes functions supplied to Agent Framework. The framework builds function schemas from type hints and descriptions, dispatches calls, and sends their results back to the model.

- Give each function one job. Describe its parameters so the model can construct valid arguments.
- `@tool` can set the name, description, schema, and approval behavior. `never_require` runs without an approval exchange; `always_require` requires the application to collect a decision and resume the run.
- The Learn lesson recommends returning a useful error string for expected tool failures so the model can explain them. Handle those failures deliberately; do not report success when the operation failed.
- One agent can act as another agent's tool. The caller receives the result and keeps control; transferring the conversation to a specialist is the [handoff pattern](../009-orchestrate-semantic-kernel-multi-agent-solution/07-use-handoff-orchestration.md).

## Lab: expense claim agent

Continue the [program and environment from unit 03](03-create-azure-ai-agent.md#one-run), using the [official expense exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html). The changes below turn its one-off draft into an email-printing workflow. The tool simulates submission; it sends no email.

### Supply the expenses

Save the exercise's [data.txt](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/data.txt) beside `agent-framework.py`:

```text
date,description,amount
07-Mar-2025,taxi,24.00
07-Mar-2025,dinner,65.50
07-Mar-2025,hotel,125.90
```

The file supplies the facts; the user's prompt supplies the requested action. Keeping those separate lets the same agent draft, summarize, or submit the supplied claim.

### Define the application function

Add these imports and the function **above `main()`** in `agent-framework.py`, keeping the imports already in unit 03:

```python
from pathlib import Path
from typing import Annotated
from agent_framework import tool
from pydantic import Field

@tool(approval_mode="never_require")
def submit_claim(
    to: Annotated[str, Field(description="The claim email recipient")],
    subject: Annotated[str, Field(description="The email subject")],
    body: Annotated[str, Field(description="Itemized expenses and their total")],
) -> str:
    """Print an expense claim email without sending it."""
    print("To:", to)
    print("Subject:", subject)
    print(body)
    return "Claim email printed. No email was sent."
```

The schema exposes three string arguments. The model fills them; the framework calls `submit_claim`; the returned string becomes tool-result context for the model's final reply. Returning a result and explicitly describing the simulation are teaching adaptations of the exercise's print-only function.

### Give the agent the tool and the data

Replace **only `main()`** from unit 03 with the following. Keep the `if __name__ == "__main__"` entry point at the bottom, after the tool definition and `main()`.

```python
async def main():
    load_dotenv()
    expenses = Path(__file__).with_name("data.txt").read_text(encoding="utf-8")
    print(expenses)
    prompt = input("What would you like me to do with these expenses? ")
    with AzureCliCredential() as credential:
        client = FoundryChatClient(
            project_endpoint=os.environ["PROJECT_ENDPOINT"],
            model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
            credential=credential,
        )
        async with Agent(
            client=client,
            name="ExpenseClaimAgent",
            instructions=(
                "Prepare expense claims using only the supplied data. "
                "When asked to submit, use submit_claim to print an email "
                "to expenses@contoso.com with subject 'Expense Claim' "
                "and a body containing itemized expenses and the total. "
                "Do not ask for more information or claim an email was sent."
            ),
            tools=[submit_claim],
        ) as agent:
            response = await agent.run(f"Request: {prompt}\nExpenses:\n{expenses}")
            print(response.text)
```

`Path(__file__).with_name(...)` resolves the data relative to the program, so running it from a different working directory still finds the file. `tools=[submit_claim]` registers the function; naming it in instructions alone would not. This run needs no manual `requires_action` loop because Agent Framework performs function dispatch.

Run `python agent-framework.py` and enter `Submit a claim for these expenses.` Look for the function's `To:` and `Subject:` output, then the model's final confirmation. Verify the three amounts sum to **215.40**. This tests both tool execution and the generated claim; a friendly final sentence alone does not prove the tool ran.

`never_require` is suitable for this local print simulation. If you replace it with an actual sending operation, choose an approval policy and implement its approval/resume flow. When finished with the Azure exercise, remove the disposable agents and model/project resources you created; closing the Python client only releases local connections.
