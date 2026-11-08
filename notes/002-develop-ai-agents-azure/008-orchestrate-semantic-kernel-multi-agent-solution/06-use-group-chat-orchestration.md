# 06 Use group chat orchestration

[Use group chat orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/6-use-group-chat-orchestration?pivots=text)

- Agents share conversation context and a manager selects the next speaker. This is turn-based discussion; whether an agent can change a live system depends on its tools and permissions.
- Use it for brainstorming, a debate, maker-checker (one writes, one reviews, repeat), or a review you can audit because it is one transcript.
- A straight pipeline usually needs less coordination. More participants increase context and coordination costs; about three is a design guideline, not an SDK limit. Configure a clear termination rule.
- In the pinned current SDK, use `GroupChatBuilder` with an `orchestrator_agent`, a `selection_func`, or an implementation of `BaseGroupChatOrchestrator`. Choose one manager configuration. `termination_condition` and `max_rounds` bound execution.
- The Learn page's `GroupChatManager` lifecycle methods (`should_request_user_input`, `should_terminate`, `filter_results`, `select_next_agent`) describe an earlier API. Do not copy those methods into the current builder without adapting them.

The Learn page copies the concurrent steps and says the agents run in parallel. They do not. The manager picks one speaker per turn.

## Maker-checker with an agent manager

In the current SDK the manager is an agent passed as `orchestrator_agent`. `termination_condition` and `max_rounds` provide limits if the manager keeps going. The output is the whole transcript. Pattern from the [Agent Framework orchestration samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/orchestrations).

Use the pinned packages and `.env` from [09-exercise.md](09-exercise.md). This is a supplementary pattern example; the official lab itself implements the sequential feedback pipeline.

```python
import asyncio
import os
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import GroupChatBuilder
from azure.identity import AzureCliCredential
from dotenv import load_dotenv

async def main():
    load_dotenv()
    client = FoundryChatClient(
        project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],  # deployment name
        credential=AzureCliCredential(),
    )
    manager = Agent(
        client=client,
        name="Manager",
        description="Picks the next speaker",
        instructions="Writer drafts first, then Reviewer checks it. Finish when Reviewer has no changes.",
    )
    writer = Agent(client=client, name="Writer", description="Drafts the reply", instructions="Draft a short reply to the support ticket.")
    reviewer = Agent(client=client, name="Reviewer", description="Checks the draft", instructions="Say what the draft got wrong, or say it is ready.")
    workflow = GroupChatBuilder(
        participants=[writer, reviewer],
        orchestrator_agent=manager,
        max_rounds=4,
        termination_condition=lambda messages: sum(1 for m in messages if m.role == "assistant") >= 4,
    ).build()
    result = await workflow.run("Users can't reset their password from the mobile app.")
    for transcript in result.get_outputs():
        if isinstance(transcript, list):
            for message in transcript:
                print(message.author_name or message.role, message.text)

asyncio.run(main())
```
