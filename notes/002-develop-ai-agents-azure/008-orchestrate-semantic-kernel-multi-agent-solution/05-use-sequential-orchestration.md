# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- The output of one agent is the input of the next. The order is fixed.
- Use it for a pipeline: classify, then draft, then check policy.
- Skip it when the steps are independent, a later step must sometimes be skipped, or a person has to approve in the middle.
- `SequentialBuilder(participants=[...]).build()`. Read `WorkflowOutputEvent` for the final result. The last agent sees the earlier replies.

## Classify, then answer

Lab: [Develop a multi-agent solution](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/08-agent-framework-multi-agents.html). `as_agent` builds each participant. `output_from="all"` keeps every agent's message.

```python
import asyncio
import os
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import SequentialBuilder
from azure.identity import AzureCliCredential

async def main():
    chat_client = FoundryChatClient(
        credential=AzureCliCredential(),
        project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
        model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),  # deployment name
    )
    summarizer_agent = chat_client.as_agent(
        name="summarizer",
        instructions="Summarize the customer feedback in two sentences.",
    )
    classifier_agent = chat_client.as_agent(
        name="classifier",
        instructions="Label the feedback as bug, feature, or praise.",
    )
    action_agent = chat_client.as_agent(
        name="action",
        instructions="Recommend one product change from the summary and the label.",
    )
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    feedback = "The dashboard works, but the bright screen hurts my eyes at night. Add a dark mode."
    result = await workflow.run(f"Customer feedback: {feedback}")
    print(result.get_outputs())

asyncio.run(main())
```
