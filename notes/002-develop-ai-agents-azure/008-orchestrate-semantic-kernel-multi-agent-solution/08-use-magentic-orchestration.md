# 08 Use Magentic orchestration

[Use Magentic orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/8-use-magentic-orchestration?pivots=text)

- A Magentic manager plans, picks the next specialist, and keeps a task ledger of goals and steps. The plan is part of the result, so a person can read it.
- Use it when the path is not known, several specialists plus tools must shape the answer, and you want the plan written down.
- Skip it when the path is fixed, you do not need a ledger, a lighter pattern works, or you need speed. Planning is slower, and a stall loop is a real failure.
- `MagenticBuilder` plus a manager agent. `max_round_count`, `max_stall_count`, and `max_reset_count` bound the loop. Run with `run_stream` and read `WorkflowOutputEvent`.

## Plan, then delegate

```python
import asyncio
from agent_framework import Agent, MagenticBuilder
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

async def main():
    client = FoundryChatClient(
        credential=DefaultAzureCredential(),
        project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
        model="gpt-4.1",  # deployment name
    )
    manager = Agent(client=client, name="Manager", instructions="Write the plan, then pick one specialist per step.")
    researcher = Agent(client=client, name="Researcher", instructions="Collect facts. Do not decide.")
    writer = Agent(client=client, name="Writer", instructions="Turn the plan and the facts into the answer.")
    workflow = MagenticBuilder(
        participants=[researcher, writer],
        manager_agent=manager,
        max_round_count=10,
        max_stall_count=3,
        max_reset_count=2,
    ).build()
    async for event in workflow.run_stream("Plan a staged rollout of the new portal."):
        if type(event).__name__ == "WorkflowOutputEvent":
            print(event)

asyncio.run(main())
```
