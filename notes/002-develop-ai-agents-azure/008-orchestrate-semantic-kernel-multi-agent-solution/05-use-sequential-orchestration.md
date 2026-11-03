# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- The output of one agent is the input of the next. The order is fixed.
- Use it for a pipeline: classify, then draft, then check policy.
- Skip it when the steps are independent, a later step must sometimes be skipped, or a person has to approve in the middle.
- `SequentialBuilder(participants=[...]).build()`. Read `WorkflowOutputEvent` for the final result. The last agent sees the earlier replies.

## Classify, then answer

```python
import asyncio
from agent_framework import Agent, SequentialBuilder
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

async def main():
    client = FoundryChatClient(
        credential=DefaultAzureCredential(),
        project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
        model="gpt-4.1",  # deployment name
    )
    classifier = Agent(client=client, name="Classifier", instructions="Label the ticket billing or technical. One word.")
    resolver = Agent(client=client, name="Resolver", instructions="Answer the ticket using the label you were given.")
    workflow = SequentialBuilder(participants=[classifier, resolver]).build()
    events = await workflow.run("I was charged twice for the same license.")
    print(events.get_outputs())

asyncio.run(main())
```
