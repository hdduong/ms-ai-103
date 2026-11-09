# 03 Implement an agent executor

[Implement an agent executor](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/3-implement-agent-executor?pivots=text)

- `AgentExecutor` is the protocol side of your logic. It does not decide discovery. The card does that.
- `execute` reads `RequestContext`, runs your code, and puts messages, task updates, or artifacts on the `EventQueue`.
- `cancel` stops a running task. A simple agent refuses it with `TaskNotCancelableError`.
- Flow: request comes in, executor calls your helper, the result becomes an event, the server sends the event back.

## Lab: the title agent's logic

From the [A2A lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). `title_agent/agent.py` is the helper: one Foundry agent, a new thread per request.

```python
import os
from azure.ai.agents import AgentsClient
from azure.ai.agents.models import ListSortOrder, MessageRole
from azure.identity import DefaultAzureCredential

class TitleAgent:
    def __init__(self):
        self.client = AgentsClient(
            endpoint=os.environ["PROJECT_ENDPOINT"],
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True,
            ),
        )
        self.agent = self.client.create_agent(
            model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
            name="title-agent",
            instructions="You are a helpful writing assistant. Given a topic the user wants to write about, suggest a single clear and catchy blog post title.",
        )

    async def run_conversation(self, user_message: str) -> list[str]:
        thread = self.client.threads.create()
        self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
        run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
        if run.status == "failed":
            raise RuntimeError(f"Title run failed: {run.last_error}")
        messages = self.client.messages.list(thread_id=thread.id, order=ListSortOrder.DESCENDING)
        for message in messages:
            if message.role == MessageRole.AGENT and message.text_messages:
                return [part.text.value for part in message.text_messages]
        return ["No title was generated."]
```

`title_agent/agent_executor.py` turns that into A2A task events: submitted, working, then completed or failed.

```python
from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events.event_queue import EventQueue
from a2a.server.tasks import TaskUpdater
from a2a.types import AgentCard, TaskNotCancelableError, TaskState
from a2a.utils import new_agent_text_message
from a2a.utils.errors import ServerError
from title_agent.agent import TitleAgent

class FoundryAgentExecutor(AgentExecutor):
    def __init__(self, card: AgentCard):
        self._card = card
        self._foundry_agent = None

    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        updater = TaskUpdater(event_queue, context.task_id, context.context_id)
        await updater.submit()
        await updater.start_work()
        try:
            if self._foundry_agent is None:
                self._foundry_agent = TitleAgent()
            await updater.update_status(
                TaskState.working,
                message=new_agent_text_message("Title Agent is processing your request...", context_id=context.context_id),
            )
            responses = await self._foundry_agent.run_conversation(context.get_user_input())
            await updater.complete(
                message=new_agent_text_message(responses[-1], context_id=context.context_id),
            )
        except Exception as e:
            await updater.failed(
                message=new_agent_text_message(f"Title agent failed: {e}", context_id=context.context_id),
            )

    async def cancel(self, context: RequestContext, event_queue: EventQueue) -> None:
        raise ServerError(error=TaskNotCancelableError())

def create_foundry_agent_executor(card: AgentCard) -> FoundryAgentExecutor:
    return FoundryAgentExecutor(card)
```

- Task state is A2A's view. It does not hand the Azure thread to the caller.
- The Azure calls are synchronous inside `async` methods, so they still block. Fine for the lab, not for a busy server.
