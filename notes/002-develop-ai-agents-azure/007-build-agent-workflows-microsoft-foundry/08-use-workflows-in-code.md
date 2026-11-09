# 08 Use workflows in code

[Use workflows in code](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/8-use-workflows-in-code?pivots=text)

A client invokes an existing workflow by name and supplies its input. The saved graph runs in the service; the Python application reads the resulting stream. A human-input workflow must keep its conversation so a later user message can resume it.

## Invoke

Use the `ContosoPay-Customer-Support-Triage` workflow built across [units 04–06](04-create-workflows-microsoft-foundry.md#first-nodes-in-the-exercise), then follow this completed adaptation of the [official client](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/06-build-workflow-ms-foundry/Python/workflow.py).

### Connect to the saved project

In a Python **3.13** working folder, create an environment and install the [exercise dependencies](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/06-build-workflow-ms-foundry/Python/requirements.txt):

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install python-dotenv azure-identity "azure-ai-projects==2.3.0" aiohttp "openai<3"
az login
```

On PowerShell, activate with `.\.venv\Scripts\Activate.ps1`. The exercise caps `openai` below 3 for compatibility with this Azure AI Projects version. Create `.env` beside the program:

```dotenv
PROJECT_ENDPOINT=https://<resource-name>.services.ai.azure.com/api/projects/<project-name>
```

Copy the project endpoint from the workflow's **Code > .env variables** panel. The portal calls the value `AZURE_EXISTING_AIPROJECT_ENDPOINT`; the program below reads it as `PROJECT_ENDPOINT`.

### Start the run and read its output

Save this complete program as `invoke_workflow.py`:

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

def main():
    load_dotenv()
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(
            endpoint=os.environ["PROJECT_ENDPOINT"],
            credential=credential,
        ) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
        conversation = openai_client.conversations.create()
        try:
            stream = openai_client.responses.create(
                conversation=conversation.id,
                extra_body={
                    "agent_reference": {
                        "name": "ContosoPay-Customer-Support-Triage",
                        "type": "agent_reference",
                    }
                },
                input="Start",
                stream=True,
            )
            for event in stream:
                if event.type == "response.completed":
                    response = openai_client.responses.retrieve(event.response.id)
                    print(response.output_text)
                elif event.type == "response.failed":
                    raise RuntimeError(f"Workflow response failed: {event.response.error}")
        finally:
            openai_client.conversations.delete(conversation_id=conversation.id)

if __name__ == "__main__":
    main()
```

```bash
python invoke_workflow.py
```

The request fields connect directly to the workflow concepts:

| Field or operation | Why it is needed |
| --- | --- |
| `conversation.id` | Groups this run's messages and state |
| `agent_reference.name` | Selects the saved workflow; it does not create its nodes or pin a version |
| `input="Start"` | Triggers this example because the tickets are already stored in `Local.SupportTickets` |
| `stream=True` | Receives events while the workflow runs |
| `response.completed` | Identifies the completed response whose text the client prints |
| Conversation deletion in `finally` | Removes this disposable run's conversation even if reading the response fails |

The [Learn unit](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/8-use-workflows-in-code?pivots=text) shows `extra_body["agent"]`; the linked exercise uses `extra_body["agent_reference"]` with `azure-ai-projects==2.3.0`. This program follows the exercise's request shape. Keep the fields and package generation together.

Check the run against the [branch table](06-apply-power-fx.md#lab-branch-conditions): every ticket should reach one route, Billing should bypass resolution, and other sufficiently confident classifications should receive a draft. Wording and confidence can vary.

This one-shot client deletes its conversation after the stream. A workflow that pauses for a user's reply must retain that conversation until the interaction is finished. Closing this client leaves workflow definitions, agents, model deployments, and the project in Azure. Remove disposable resources when finished, and export YAML before the [designer retirement](02-understand-workflows.md).
