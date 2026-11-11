# 03 Create an Azure AI agent with Microsoft Agent Framework

[Create an Azure AI agent with Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/3-create-azure-ai-agent?pivots=text)

An agent needs a model connection and instructions before it can do useful work. The expense-claim scenario from [Microsoft Learn](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/07-agent-framework.html) makes the difference visible: this unit drafts a claim; the next unit adds a function that prints the prepared email.

- Use the **project endpoint** and your **deployment name**. A model catalog name alone does not select a deployed resource.
- The exercise uses `AzureCliCredential`, which reads the account selected by `az login`. `DefaultAzureCredential` is a credential chain that can also use developer credentials or managed identity, depending on its environment.
- `Agent.run()` may include several model/tool exchanges before producing one response. For multiple user turns, pass the same session as explained in [unit 02](02-understand-semantic-kernel-agents.md#where-conversation-state-lives).

## One run

Create a working folder, use Python **3.13** as in the official exercise, and install its [package versions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/requirements.txt):

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install python-dotenv azure-identity==1.25.3 "pydantic>=2,<3" agent-framework-core==1.12.1 agent-framework-foundry==1.10.3
az login
```

On PowerShell, activate with `.\.venv\Scripts\Activate.ps1`. Create a Foundry project and deploy a tool-capable model such as the exercise's `gpt-5`. Save its values in `.env` beside the program:

```dotenv
PROJECT_ENDPOINT=https://<resource-name>.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

Save the following as `agent-framework.py`. This is a complete, smaller version of the exercise's agent creation step; it deliberately has no custom tool yet.

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from dotenv import load_dotenv

async def main():
    load_dotenv()
    with AzureCliCredential() as credential:
        client = FoundryChatClient(
            project_endpoint=os.environ["PROJECT_ENDPOINT"],
            model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
            credential=credential,
        )
        async with Agent(
            client=client,
            name="ExpenseClaimAgent",
            instructions="Draft an expense claim email from the supplied expenses. Do not claim to send it.",
        ) as agent:
            response = await agent.run("Draft a claim for taxi 42 and lunch 18.")
            print(response.text)

if __name__ == "__main__":
    asyncio.run(main())
```

```bash
python agent-framework.py
```

`FoundryChatClient` supplies authentication, project, and model configuration. `Agent` supplies the task instructions. `await agent.run(...)` waits for the model's completed response; `response.text` extracts the text to display. Context managers release local client resources when the run finishes.

Check that the draft lists both expenses and totals **60**. Wording can vary. At this point the application only prints the model response; [unit 04](04-add-plugins-to-agent.md#lab-expense-claim-agent) connects the model to a Python function and real input data.

The framework can read `FOUNDRY_PROJECT_ENDPOINT` and `FOUNDRY_MODEL` automatically. This exercise uses its own variable names and passes them explicitly, so keep the names in `.env` and `os.environ` consistent.
