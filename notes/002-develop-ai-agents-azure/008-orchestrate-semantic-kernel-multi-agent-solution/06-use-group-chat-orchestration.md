# 06 Use group chat orchestration

[Use group chat orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/6-use-group-chat-orchestration?pivots=text)

- Agents share one thread. A chat manager chooses who speaks and when a person should join. Agents talk. They do not change live systems.
- Use it for brainstorming, a debate, maker-checker (one writes, one reviews, repeat), or a review you can audit because it is one transcript.
- Skip it for a straight pipeline, when you need a fast answer, or with more than about three agents. The manager has to know when to stop.
- `GroupChatBuilder`. A custom `GroupChatManager` can change the rules.
- Each round, in order: `should_request_user_input`, `should_terminate`, then either `filter_results` or `select_next_agent`.

The Learn page copies the concurrent steps and says the agents run in parallel. They do not. The manager picks one speaker per turn.

## Three turns, then stop

```python
import asyncio
from agent_framework import Agent, GroupChatBuilder
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential

async def main():
    client = FoundryChatClient(
        credential=DefaultAzureCredential(),
        project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
        model="gpt-4.1",  # deployment name
    )
    manager = Agent(client=client, name="Manager", instructions="Pick the next speaker. Stop when the reply is good enough.")
    writer = Agent(client=client, name="Writer", instructions="Draft a short reply to the ticket.")
    reviewer = Agent(client=client, name="Reviewer", instructions="Say what the draft got wrong.")
    workflow = GroupChatBuilder(
        participants=[writer, reviewer],
        orchestrator_agent=manager,
    ).with_max_rounds(3).build()
    events = await workflow.run("Users can't reset their password from the mobile app.")
    print(events.get_outputs())

asyncio.run(main())
```
