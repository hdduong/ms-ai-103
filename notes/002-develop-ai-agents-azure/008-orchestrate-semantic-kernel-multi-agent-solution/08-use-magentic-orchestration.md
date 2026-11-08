# 08 Use Magentic orchestration

[Use Magentic orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/8-use-magentic-orchestration?pivots=text)

- The manager maintains a task/progress ledger and directs specialists. It tracks what is known, what remains, and whether progress stalled. Do not assume the final answer includes the internal plan: use workflow observability or explicit plan review when you need to inspect it.
- Use it when the path is not known, several specialists plus tools must shape the answer, and you want the plan written down.
- Skip it when the path is fixed, you do not need a ledger, a lighter pattern works, or you need speed. Planning is slower, and a stall loop is a real failure.
- `MagenticBuilder` plus a manager agent. `max_round_count`, `max_stall_count`, and `max_reset_count` bound the loop. The Learn page says `run_stream`. In the current SDK that is `run(task, stream=True)`, or `run(task)` and then `get_outputs()`.

## Plan, then delegate

Pattern from the [Agent Framework orchestration samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/orchestrations). The coder gets the hosted code interpreter, so the manager has a specialist for math. The researcher in this example has no search tool; it extracts supplied assumptions and cannot verify current market prices. The prompt therefore supplies illustrative rates.

Use the pinned packages and `.env` from [09-exercise.md](09-exercise.md). This is a supplementary pattern example; the official lab itself implements the sequential feedback pipeline.

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import MagenticBuilder
from azure.identity import AzureCliCredential
from dotenv import load_dotenv

async def main():
    load_dotenv()
    client = FoundryChatClient(
        project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],  # deployment name
        credential=AzureCliCredential(),
    )
    researcher = Agent(
        client=client,
        name="ResearcherAgent",
        description="Extracts the supplied assumptions",
        instructions="Extract the assumptions supplied in the task. Do not invent current prices or browse; no search tool is configured. Do not calculate.",
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
    result = await workflow.run(
        "Estimate a 30-day token cost for 10,000 chats/day, each with "
        "1,000 input tokens and 500 output tokens. Use hypothetical rates "
        "of $2/million input tokens and $8/million output tokens. "
        "Show assumptions and arithmetic; these are not current model prices."
    )
    for output in result.get_outputs():
        print(output.text)

asyncio.run(main())
```
