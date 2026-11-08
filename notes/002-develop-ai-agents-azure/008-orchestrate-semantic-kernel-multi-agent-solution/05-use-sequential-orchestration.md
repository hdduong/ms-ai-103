# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- Agents run in a fixed order. The default builder forwards the accumulated conversation, including earlier replies, to the next participant; it does not pass only the previous agent's final string.
- Use it for a pipeline: classify, then draft, then check policy.
- Use concurrent orchestration for independent steps. If steps must branch or pause for approval, design those edges or approval steps explicitly instead of assuming a plain sequential list provides them.
- `SequentialBuilder(participants=[...]).build()` creates the workflow. Use `result.get_outputs()` after a non-streaming run, or events with `event.type == "output"` in a streaming run.

## Classify, then answer

Lab: [Develop a multi-agent solution](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/08-agent-framework-multi-agents.html). `as_agent` builds each participant. `output_from="all"` emits each participant's response. Use the pinned dependencies and `.env` in [09-exercise.md](09-exercise.md).

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
