# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- The output of one agent is the input of the next. The order is fixed. In code the builder passes the whole conversation so far, so a later agent also sees the earlier replies.
- Use it for a pipeline: classify, then draft, then check policy.
- Skip it when the steps are independent (use concurrent), a later step must sometimes be skipped, or a person has to approve in the middle.
- `SequentialBuilder(participants=[...]).build()` fixes the participant order. The packages below expose results through `result.get_outputs()`, or `event.type == "output"` when streaming. The Learn unit calls the output event `WorkflowOutputEvent`.

## Lab: feedback pipeline

Lab: [Develop a multi-agent solution](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/08-agent-framework-multi-agents.html). Summarize, classify, then suggest an action. `as_agent` builds each participant. `output_from="all"` keeps every agent's message; it does not make them run in parallel.

### Prepare the program

Use Python **3.13** and a deployed model in a Foundry project. In a working folder, create an environment and install the [exercise package versions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/08-agent-orchestration/Python/requirements.txt):

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install python-dotenv azure-identity==1.25.3 agent-framework-core==1.12.1 agent-framework-foundry==1.10.3 agent-framework-orchestrations==1.0.1
az login
```

On PowerShell, activate with `.\.venv\Scripts\Activate.ps1`. Save `.env` beside the program. This lab uses `AZURE_AI_` names, not the ones from module 008:

```dotenv
AZURE_AI_PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
AZURE_AI_MODEL_DEPLOYMENT_NAME=gpt-5
```

### Connect each role to the pipeline

Save this completed, condensed adaptation as `agents.py`. Each instruction set has one responsibility:

| Agent | Receives | Adds to the conversation |
| --- | --- | --- |
| `summarizer` | Original feedback | A short neutral summary |
| `classifier` | Feedback and summary | Positive, Negative, or Feature request |
| `action` | All earlier messages | A suggested next step |

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
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],  # deployment name
    )
    summarizer_agent = chat_client.as_agent(
        name="summarizer",
        instructions="Summarize the customer's feedback in one short, neutral sentence.",
    )
    classifier_agent = chat_client.as_agent(
        name="classifier",
        instructions="Classify the feedback as exactly one of: Positive, Negative, or Feature request.",
    )
    action_agent = chat_client.as_agent(
        name="action",
        instructions="Based on the summary and classification, suggest the next action in one short sentence.",
    )
    feedback = (
        "I use the dashboard every day to monitor metrics, and it works well overall. "
        "But when I'm working late at night, the bright screen is really harsh on my eyes. "
        "If you added a dark mode option, it would make the experience much more comfortable."
    )
    # List order defines execution order; later agents see accumulated messages.
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    result = await workflow.run(f"Customer feedback: {feedback}")
    for response in result.get_outputs():
        for message in response.messages:
            print(f"[{message.author_name or message.role}] {message.text}")

if __name__ == "__main__":
    asyncio.run(main())
```

`as_agent(...)` creates agents sharing one client configuration while keeping separate roles. `SequentialBuilder` passes the accumulated conversation onward, so the action agent has the original feedback, summary, and classification. `output_from="all"` exposes each participant response; the nested print loop reads each response's messages and labels their authors.

### Check the result

```bash
python agents.py
```

For the supplied prompt, expect a neutral summary of the dark-mode request, a `Feature request` classification, and an action such as adding it to the backlog. Exact wording can vary.

Replace `feedback` with either of these exercise cases and rerun:

| Input | Classification and useful action to check |
| --- | --- |
| `The app crashes when I upload a photo.` | Negative; investigate the upload failure |
| `I love the new dark mode. It makes the app easier to use at night.` | Positive; record or share the positive feedback |

The participant order stays summarizer, classifier, action for every input. If a business process needs to skip or select a step based on classification, add a conditional route as in [unit 07](07-use-handoff-orchestration.md), rather than expecting a sequential builder to infer one.
