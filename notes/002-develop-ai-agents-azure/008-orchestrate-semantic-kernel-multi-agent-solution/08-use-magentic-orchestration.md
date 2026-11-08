# 08 Use Magentic orchestration

[Use Magentic orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/8-use-magentic-orchestration?pivots=text)

- A Magentic manager plans, picks the next specialist, and keeps a task ledger of goals and steps. The plan is part of the result, so a person can read it.
- Use it when the path is not known, several specialists plus tools must shape the answer, and you want the plan written down.
- Skip it when the path is fixed, you do not need a ledger, a lighter pattern works, or you need speed. Planning is slower, and a stall loop is a real failure.
- `MagenticBuilder` plus a manager agent. `max_round_count`, `max_stall_count`, and `max_reset_count` bound the loop. The Learn page says `run_stream`. In the current SDK that is `run(task, stream=True)`, or `run(task)` and then `get_outputs()`.

## Plan, then delegate

Pattern from the [Agent Framework orchestration samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/orchestrations). The coder gets the hosted code interpreter, so the manager has a specialist for math.

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import MagenticBuilder
from azure.identity import AzureCliCredential

async def main():
    client = FoundryChatClient(
        project_endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
        model=os.environ["FOUNDRY_MODEL"],  # deployment name
        credential=AzureCliCredential(),
    )
    researcher = Agent(
        client=client,
        name="ResearcherAgent",
        description="Finds information",
        instructions="Find facts. Do not calculate.",
    )
    coder = Agent(
        client=client,
        name="CoderAgent",
        description="Writes and runs code",
        instructions="Answer with code and show the calculation.",
        tools=client.get_code_interpreter_tool(),
    )
    manager = Agent(
        client=client,
        name="MagenticManager",
        description="Plans and delegates",
        instructions="Plan the steps, pick one specialist per step, and stop when the task is done.",
    )
    workflow = MagenticBuilder(
        participants=[researcher, coder],
        manager_agent=manager,
        max_round_count=10,
        max_stall_count=3,
        max_reset_count=2,
    ).build()
    result = await workflow.run("Estimate the monthly token cost of 10,000 chats a day at 1,500 tokens each.")
    for output in result.get_outputs():
        print(output.text)

asyncio.run(main())
```
