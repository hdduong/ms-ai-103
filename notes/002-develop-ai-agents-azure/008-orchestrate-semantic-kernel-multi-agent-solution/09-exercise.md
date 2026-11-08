# 09 Exercise — Orchestrate three feedback agents

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/9-exercise) · [Official lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/08-agent-framework-multi-agents.html)

Reviewed **2026-11-07** against [the official instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/08-agent-framework-multi-agents.md), [starter program](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/08-agent-orchestration/Python/agents.py), and [requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/08-agent-orchestration/Python/requirements.txt) at commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`. The example below is a completed, condensed adaptation; see [MIT attribution](../../../THIRD_PARTY_NOTICES.md).

## What you build

The exercise implements **sequential orchestration**. A summarizer reads customer feedback; a classifier labels it `Positive`, `Negative`, or `Feature request`; an action agent recommends one next step. All three use the same Foundry client but have separate instructions and names. The fixed participant order controls execution.

The official hands-on exercise does not implement all five orchestration patterns covered by this module. [Concurrent](04-use-concurrent-orchestration.md), [group chat](06-use-group-chat-orchestration.md), [handoff](07-use-handoff-orchestration.md), and [Magentic](08-use-magentic-orchestration.md) remain supplementary examples linked from their lessons.

## 1. Set up the lab

Use the official lab's Python **3.13** environment (tested with **3.13.12**), Azure project, Foundry Toolkit, and a deployed `gpt-5` model where available. Copy your actual project endpoint and deployment name.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
git -C mslearn-ai-agents checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd mslearn-ai-agents/Labfiles/08-agent-orchestration/Python
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

Use `.\.venv\Scripts\Activate.ps1` for PowerShell activation. Direct dependency versions from the lab:

```text
python-dotenv
azure-identity==1.25.3
agent-framework-core==1.12.1
agent-framework-foundry==1.10.3
agent-framework-orchestrations==1.0.1
```

The orchestration builders are imported from `agent_framework.orchestrations` and require the additional `agent-framework-orchestrations` package. These are direct dependency pins; upstream does not supply a complete transitive lockfile here.

Use these exact local `.env` names:

```dotenv
AZURE_AI_PROJECT_ENDPOINT=https://YOUR-RESOURCE.services.ai.azure.com/api/projects/YOUR-PROJECT
AZURE_AI_MODEL_DEPLOYMENT_NAME=YOUR-DEPLOYMENT-NAME
```

The lab prose mentions `MODEL_DEPLOYMENT_NAME` in one setup step, but its checked-in `.env` and code use the `AZURE_AI_` names above. These differ from module 007. This program passes their values explicitly, so similarly named settings will not substitute for them. Do not commit local configuration.

## 2. Complete the sequential program

Save as `feedback_agents.py` alongside `.env`:

```python
import asyncio
import os

from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import SequentialBuilder
from azure.identity import AzureCliCredential
from dotenv import load_dotenv


async def main():
    load_dotenv()
    chat_client = FoundryChatClient(
        credential=AzureCliCredential(),
        project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    )
    summarizer = chat_client.as_agent(
        name="summarizer",
        instructions="Summarize the customer's feedback in one short, neutral sentence.",
    )
    classifier = chat_client.as_agent(
        name="classifier",
        instructions=(
            "Classify the feedback as exactly one of: "
            "Positive, Negative, or Feature request."
        ),
    )
    action = chat_client.as_agent(
        name="action",
        instructions=(
            "Based on the summary and classification, suggest the next action "
            "in one short sentence."
        ),
    )
    feedback = (
        "I use the dashboard every day to monitor metrics, and it works well overall. "
        "But when I'm working late at night, the bright screen is really harsh on my eyes. "
        "If you added a dark mode option, it would make the experience much more comfortable."
    )

    async with summarizer, classifier, action:
        workflow = SequentialBuilder(
            participants=[summarizer, classifier, action],
            output_from="all",
        ).build()
        result = await workflow.run(f"Customer feedback: {feedback}")
        for response in result.get_outputs():
            for message in response.messages:
                print(f"[{message.author_name or message.role}] {message.text}")


if __name__ == "__main__":
    asyncio.run(main())
```

```bash
python feedback_agents.py
```

This adaptation adds explicit agent context managers and uses shorter instructions while retaining the official client, agents, builder, and output-processing pattern. The default sequential builder passes the accumulated conversation to later agents; it is not restricted to handing over only the previous response's text. `output_from="all"` exposes every participant's output for inspection; it does not make execution concurrent.

## 3. Check behavior

The dark-mode feedback should produce a concise summary, `Feature request`, and a recommendation to log or prioritize a dark-mode enhancement. Generated wording is variable. Try the official lab's alternatives: a crash during photo upload and praise for a feature. Check that the three roles still run in the declared order and that the recommendation follows the classification.

For an application that branches on the classifier's result, replace free-text labels with a validated schema and handle invalid outputs explicitly. This exercise prints recommendations; it does not create issues or modify a product backlog.

## Current API differences to recognize

The Learn conceptual units and the current lab do not always use the same SDK generation. For this exercise's `core==1.12.1` / `orchestrations==1.0.1` baseline:

| Older text you may encounter | Form used by this baseline |
| --- | --- |
| `chat_client.create_agent(...)` | `chat_client.as_agent(...)`, or construct `Agent(client=...)`. |
| Builder `.participants([...])` shown for every pattern | `SequentialBuilder(participants=[...], output_from="all").build()`. Handoff still has its own fluent configuration methods. |
| `workflow.run_stream(...)` | `workflow.run(..., stream=True)` for an async event stream. |
| Separate `WorkflowOutputEvent` / `RequestInfoEvent` classes | A `WorkflowEvent` with a `type` discriminator, such as `"output"` or `"request_info"`. |
| Group chat described as “all agents concurrently” | Group chat selects the next speaker; concurrent orchestration is a distinct pattern. |

These differences were checked against the published [core 1.12.1](https://pypi.org/project/agent-framework-core/1.12.1/) and [orchestrations 1.0.1](https://pypi.org/project/agent-framework-orchestrations/1.0.1/) package source. For changes beyond this baseline, follow the [official SDK samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows), using the matching package versions.

## Cleanup

Close the client process and remove dedicated model deployments or the dedicated resource group when finished. Agent context managers do not delete your Azure project or model deployments. All three agents consume the same deployment's quota; sequential execution still makes multiple inference calls.

**Validation:** the Python program was syntax checked and the constructors, builder arguments, and output APIs were checked against the pinned package source. It was not executed against Azure during this review.
