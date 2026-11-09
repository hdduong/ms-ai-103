# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- The output of one agent is the input of the next. The order is fixed. In code the builder passes the whole conversation so far, so a later agent also sees the earlier replies.
- Use it for a pipeline: classify, then draft, then check policy.
- Skip it when the steps are independent (use concurrent), a later step must sometimes be skipped, or a person has to approve in the middle.
- `SequentialBuilder(participants=[...]).build()`. Read `WorkflowOutputEvent` for the final result. In the current SDK that is `result.get_outputs()`, or `event.type == "output"` when streaming.

## Lab: feedback pipeline

Lab: [Develop a multi-agent solution](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/08-agent-framework-multi-agents.html). Summarize, classify, then suggest an action. `as_agent` builds each participant. `output_from="all"` keeps every agent's message; it does not make them run in parallel.

```bash
pip install python-dotenv azure-identity==1.25.3 agent-framework-core==1.12.1 agent-framework-foundry==1.10.3 agent-framework-orchestrations==1.0.1
az login
```

`.env`. This lab uses `AZURE_AI_` names, not the ones from module 008:

```dotenv
AZURE_AI_PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
AZURE_AI_MODEL_DEPLOYMENT_NAME=gpt-5
```

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
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    result = await workflow.run(f"Customer feedback: {feedback}")
    for response in result.get_outputs():
        for message in response.messages:
            print(f"[{message.author_name or message.role}] {message.text}")

asyncio.run(main())
```

- Expect a one-line summary, `Feature request`, and an action such as adding dark mode to the backlog.
- Try the lab's other feedback too: a crash during photo upload, and praise for a feature. The order stays summarizer, classifier, action.
