# 06 Use group chat orchestration

[Use group chat orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/6-use-group-chat-orchestration?pivots=text)

- Agents share a conversation. A manager selects the next speaker and decides when to finish or request user input. Configured tools can still perform actions; group chat itself does not make agents read-only.
- Use it for brainstorming, a debate, maker-checker (one writes, one reviews, repeat), or a review you can audit because it is one transcript.
- Prefer a simpler pipeline when dependencies are fixed or latency matters. More participants increase coordination cost; the manager needs an explicit stopping rule.
- The Learn page describes `GroupChatManager` callbacks (`should_request_user_input`, `should_terminate`, `filter_results`, `select_next_agent`). The pinned SDK uses `orchestrator_agent`, `selection_func`, or a `BaseGroupChatOrchestrator`, plus `termination_condition` and `max_rounds`. Use the API matching your installed packages.

The Learn page copies the concurrent steps and says the agents run in parallel. They do not. The manager picks one speaker per turn.

## Maker-checker with an agent manager

In the current SDK the manager is an agent passed as `orchestrator_agent`. `termination_condition` and `max_rounds` provide limits if the manager keeps going. The output is the whole transcript. Pattern from the [Agent Framework orchestration samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/orchestrations).

Packages and `.env` as in the [lab](05-use-sequential-orchestration.md#lab-feedback-pipeline). The lab itself only builds the sequential pipeline.

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

if __name__ == "__main__":
    asyncio.run(main())
```


Save as `group_review.py`. `manager` controls turn selection; it is separate from the `participants` that draft and review. Each participant sees the shared transcript, so the reviewer can assess the writer's actual response and the writer can revise it. The manager may finish early; the explicit limits bound continued discussion.

Check the printed authors and content: Writer should draft a reply, and Reviewer should comment on that draft. This program generates and reviews text; it contains no tool that sends a support response. The transcript is useful for inspecting the collaboration, while the [sequential exercise](05-use-sequential-orchestration.md#lab-feedback-pipeline) offers a fixed-order alternative.
