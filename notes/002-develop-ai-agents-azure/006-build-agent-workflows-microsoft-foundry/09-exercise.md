# 09 Exercise — Build and invoke a Foundry workflow

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/9-exercise) · [Official lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html)

Reviewed **2026-11-07** against the [official instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/06-build-workflow-ms-foundry.md) and [Python starter](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/06-build-workflow-ms-foundry/Python/workflow.py) at commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`. Code below is a condensed, completed adaptation of the lab; Microsoft code is MIT licensed, with attribution in [THIRD_PARTY_NOTICES.md](../../../THIRD_PARTY_NOTICES.md).

## What this exercise builds

Create **ContosoPay-Customer-Support-Triage** in the Foundry designer, then call that saved workflow from Python. The workflow iterates over three tickets, asks a triage agent for structured JSON, applies deterministic Power Fx conditions, and either drafts a response or displays an escalation message. The Python client invokes the saved definition; it does not construct the canvas or create the two agents.

**Migration notice:** Microsoft's [workflow documentation](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/workflow#migration-guide) says the visual designer and in-portal workflow execution are unsupported after **December 1, 2026**. YAML definitions can continue to run when deployed as hosted agents. Export the YAML before that date; use Microsoft Agent Framework for new code-based orchestration, or Azure Logic Apps when a visual designer is needed. This lesson preserves the currently published designer exercise and its invocation API; it is not a completed hosted-agent migration.

## 1. Create the workflow in the portal

Use a Foundry project and models available to that project, then follow the official lab's designer steps. Save each change. Configure these nodes in order:

| Node | Configuration and result |
| --- | --- |
| Set variable | Store the array below as `Local.SupportTickets`. |
| For each | Iterate `Local.SupportTickets`, with loop item `Local.CurrentTicket`. |
| Invoke `Triage-Agent` | Input `Local.CurrentTicket`; text output `Local.TriageOutputText`; JSON output `Local.TriageOutputJson`. |
| If/Else | Test `Local.TriageOutputJson.confidence > 0.6`. |
| Low-confidence branch | Deliver the clarification message; this lab does not wait for an answer. |
| High-confidence branch | Test `Local.TriageOutputJson.category = "Billing"`. |
| Billing branch | Deliver an escalation message. No email, queue, or human assignment is created. |
| Other categories | Invoke `Resolution-Agent` with `Local.TriageOutputText`; save `Local.ResolutionOutputText`. |

Ticket array from the exercise:

```json
[
  "The API returns a 403 error when creating invoices, but our API key hasn't changed.",
  "Is there a way to export all invoices as a CSV?",
  "I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?"
]
```

The triage instructions require one of `Billing`, `Technical`, or `General`, plus a confidence between 0 and 1. Billing means an incorrect charge, refund, or payment; invoice export functionality is General. Under the triage agent's **Parameters > Text format > JSON Schema**, use the lab's schema:

```json
{
  "name": "category_response",
  "schema": {
    "type": "object",
    "properties": {
      "customer_issue": {"type": "string"},
      "category": {"type": "string"},
      "confidence": {"type": "number"}
    },
    "additionalProperties": false,
    "required": ["customer_issue", "category", "confidence"]
  },
  "strict": true
}
```

This schema enforces shape and types. It does **not** encode the three allowed category values or the confidence range; the lab supplies those constraints in instructions. A production schema can add an enum and bounds where supported. The model's self-reported confidence is a routing signal, not a measured probability of correctness. Evaluate the threshold against labeled tickets before treating it as a reliable automation policy.

Power Fx conditions:

```text
Local.TriageOutputJson.confidence > 0.6
Local.TriageOutputJson.category = "Billing"
```

A confidence exactly equal to `0.6` goes to the low-confidence branch because the comparison is strict. A **Deliver a message** node only displays text. A real human-in-the-loop pause uses **Ask a question** and resumes with the user's answer on the same conversation.

## 2. Set up the Python client

The upstream lab specifies Python **3.13**, tested with **3.13.12**. In a fresh clone, use the reviewed source revision and the exact case-sensitive folder name:

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
git -C mslearn-ai-agents checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd mslearn-ai-agents/Labfiles/06-build-workflow-ms-foundry/Python
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

On PowerShell, activate with `.\.venv\Scripts\Activate.ps1` instead of the `source` command. The lab's [requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/06-build-workflow-ms-foundry/Python/requirements.txt) are:

```text
python-dotenv
azure-identity
azure-ai-projects==2.3.0
aiohttp
openai<3
```

The upstream file explicitly caps `openai` below 3 because that release's HTTP client change is incompatible with this Azure AI Projects dependency. Other dependencies are unpinned upstream; this is a recorded lab baseline, not a full dependency lock.

Create or update the local `.env` file:

```dotenv
PROJECT_ENDPOINT=https://YOUR-RESOURCE.services.ai.azure.com/api/projects/YOUR-PROJECT
```

Use the **project endpoint**, copied from the portal's Code tab. The portal may label it `AZURE_EXISTING_AIPROJECT_ENDPOINT`; this lab reads the value under `PROJECT_ENDPOINT`. Keep local configuration out of source control. Your signed-in account needs permission to invoke the workflow in that project.

## 3. Invoke the saved workflow

Save this complete client as `invoke_workflow.py` beside `.env`. It preserves the exercise's request shape and simplifies its output formatter. Cleanup is in `finally` so a failed response does not skip conversation deletion.

```python
import os

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def main():
    load_dotenv()
    workflow_name = "ContosoPay-Customer-Support-Triage"

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
                        "name": workflow_name,
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
        finally:
            openai_client.conversations.delete(conversation_id=conversation.id)


if __name__ == "__main__":
    main()
```

```bash
python invoke_workflow.py
```

`"Start"` triggers this particular workflow because its ticket array is already stored in the definition. A different workflow may use the input as its actual ticket. The request references the workflow by name; it does not pin a saved workflow version.

**Source discrepancy:** Learn unit 8 shows `extra_body={"agent": ...}`; the linked exercise at the reviewed commit uses `extra_body={"agent_reference": ...}` with `azure-ai-projects==2.3.0`. The code above follows that versioned exercise. Do not combine request fields from different SDK generations without checking the corresponding current service example.

## Check the result and clean up

Check that each ticket reaches one branch, Billing bypasses the resolution agent, and output contains the triage fields plus either a drafted response or the relevant message. Generated wording and confidence values can vary. To test branch logic separately from model variability, temporarily use known test values in a separate test copy of the workflow.

The client deletes its conversation, but leaves the workflow, agents, model deployments, and project in place. Delete the dedicated lab resources when finished; delete a whole resource group only if it was created solely for this exercise. Export the workflow YAML from the portal if you intend to continue using it after designer retirement.

**Validation for these notes:** the Python block was syntax checked and its SDK names compared with the official source. The Azure workflow was not deployed or invoked during this repository review.
