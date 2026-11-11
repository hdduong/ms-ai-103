# 03 Implement an agent executor

[Implement an agent executor](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/3-implement-agent-executor?pivots=text)

The executor is the boundary between A2A requests and your agent's implementation. `execute` reads a `RequestContext`, performs the work, and puts messages or task updates on an `EventQueue`. The server turns those events into protocol responses.

The title scenario separates two responsibilities: the backend calls Foundry, and the executor reports A2A task progress. These are completed adaptations of the [official exercise files](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/title_agent).

## Lab: the title agent's logic

After [unit 02's setup](02-define-a2a-agent.md#lab-setup), replace **all of `title_agent/agent.py`** with:

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
            instructions=(
                "Given a topic for an article, suggest a single clear "
                "and catchy blog post title."
            ),
        )

    async def run_conversation(self, user_message: str) -> list[str]:
        thread = self.client.threads.create()
        try:
            self.client.messages.create(
                thread_id=thread.id, role=MessageRole.USER, content=user_message,
            )
            run = self.client.runs.create_and_process(
                thread_id=thread.id, agent_id=self.agent.id,
            )
            if run.status != "completed":
                raise RuntimeError(f"Title run ended with status {run.status}")
            messages = self.client.messages.list(
                thread_id=thread.id, order=ListSortOrder.DESCENDING,
            )
            for message in messages:
                if message.role == MessageRole.AGENT and message.text_messages:
                    return [part.text.value for part in message.text_messages]
            raise RuntimeError("The completed run returned no title text")
        finally:
            self.client.threads.delete(thread.id)
```

The constructor creates one backend agent for this server instance. Every request gets a new Azure thread, adds the user's topic, starts a run, and reads the latest assistant text. Consequently, these title requests are independent; an A2A context id does not automatically reuse an Azure thread. `finally` deletes the temporary thread after its result has been read, including when processing fails.

## A2A task events

Replace **all of `title_agent/agent_executor.py`** with:

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
                message=new_agent_text_message(
                    "Generating a title...", context_id=context.context_id,
                ),
            )
            responses = await self._foundry_agent.run_conversation(context.get_user_input())
            await updater.complete(
                message=new_agent_text_message(
                    responses[-1], context_id=context.context_id,
                ),
            )
        except Exception:
            await updater.failed(
                message=new_agent_text_message(
                    "The title agent could not complete the request.",
                    context_id=context.context_id,
                ),
            )

    async def cancel(self, context: RequestContext, event_queue: EventQueue) -> None:
        raise ServerError(error=TaskNotCancelableError())

def create_foundry_agent_executor(card: AgentCard) -> FoundryAgentExecutor:
    return FoundryAgentExecutor(card)
```

`context.get_user_input()` extracts the incoming text. `TaskUpdater` uses the A2A task and context identifiers to emit submitted, working, and completed/failed states. On success the final task message contains the generated title; on error the caller receives a failed task.

This adaptation explicitly rejects cancellation because it cannot cancel an in-flight Azure operation. Emitting a failed task with a “cancelled” message, as the starter does, would not demonstrate that backend work stopped. An implementation that supports cancellation must track and stop that operation too.

The Azure SDK calls above are synchronous even though `run_conversation` is declared `async`. They can block a busy server's event loop; this compact exercise keeps the original SDK style. An application serving concurrent traffic should use async SDK calls or move blocking work off the event loop. Next, [unit 04](04-host-a2a-agent-server.md) makes the executor available over HTTP.
