# 04 Use concurrent orchestration

[Use concurrent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/4-use-concurrent-orchestration?pivots=text)

- Every agent gets the same task and works alone. Combine the replies, or leave them as separate side effects.
- Use it for brainstorming, ensemble reasoning, a vote, or when waiting in series is too slow.
- Skip it when the next step needs the previous answer, the order must be stable, quota cannot cover parallel calls, agents would write the same data at once, or you cannot merge conflicts.
- `ConcurrentBuilder(participants=[...]).build()`, then `run`, then `get_outputs()`. Each message has an author.

## Two agents, one task

```python
import asyncio
from agent_framework import Agent, ConcurrentBuilder
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

async def main():
    client = FoundryChatClient(
        credential=DefaultAzureCredential(),
        project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
        model="gpt-4.1",  # deployment name
    )
    researcher = Agent(client=client, name="Researcher", instructions="List the facts. Do not decide.")
    writer = Agent(client=client, name="Writer", instructions="Give a one-paragraph recommendation.")
    workflow = ConcurrentBuilder(participants=[researcher, writer]).build()
    events = await workflow.run("Compare hot and cool blob tiers for a 90-day archive.")
    print(events.get_outputs())

asyncio.run(main())
```
