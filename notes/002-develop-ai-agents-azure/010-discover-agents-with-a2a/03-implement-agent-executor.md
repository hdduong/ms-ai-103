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

# Backend: plain Foundry calls, no A2A code here
class TitleAgent:
    def __init__(self):
        # Foundry project client; signs in as you via az login
        self.client = AgentsClient(
            endpoint=os.environ["PROJECT_ENDPOINT"],
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,  # ignore service principal env vars
                exclude_managed_identity_credential=True,  # no managed identity on a dev box
            ),
        )
        # Create the Foundry agent once; every request reuses it
        self.agent = self.client.create_agent(
            model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
            name="title-agent",
            instructions=(
                "Given a topic for an article, suggest a single clear "
                "and catchy blog post title."
            ),
        )

    async def run_conversation(self, user_message: str) -> list[str]:
        # A new thread per request, so requests share no history
        thread = self.client.threads.create()
        try:
            # Add the topic as the user's message
            self.client.messages.create(
                thread_id=thread.id, role=MessageRole.USER, content=user_message,
            )
            # Run the agent on the thread; blocks until the run finishes
            run = self.client.runs.create_and_process(
                thread_id=thread.id, agent_id=self.agent.id,
            )
            # Treat any run that didn't complete as an error
            if run.status != "completed":
                raise RuntimeError(f"Title run ended with status {run.status}")
            # Newest first, so the agent's reply comes before the topic
            messages = self.client.messages.list(
                thread_id=thread.id, order=ListSortOrder.DESCENDING,
            )
            # Return the text parts of the newest agent message
            for message in messages:
                if message.role == MessageRole.AGENT and message.text_messages:
                    return [part.text.value for part in message.text_messages]
            raise RuntimeError("The completed run returned no title text")
        finally:
            # Delete the temporary thread, even if the run failed
            self.client.threads.delete(thread.id)
```

Title requests are independent: an A2A context id does not automatically reuse an Azure thread.

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

# A2A side: the server calls execute() for each incoming message
class FoundryAgentExecutor(AgentExecutor):
    def __init__(self, card: AgentCard):
        self._card = card
        self._foundry_agent = None

    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        # Helper that posts this task's status events to the queue
        updater = TaskUpdater(event_queue, context.task_id, context.context_id)
        # Mark the task submitted, then working
        await updater.submit()
        await updater.start_work()
        try:
            # Create the backend on first use, so startup makes no Azure calls
            if self._foundry_agent is None:
                self._foundry_agent = TitleAgent()
            # Progress update: still working, with a short status message
            await updater.update_status(
                TaskState.working,
                message=new_agent_text_message(
                    "Generating a title...", context_id=context.context_id,
                ),
            )
            # Send the message's text to the backend and wait for the title
            responses = await self._foundry_agent.run_conversation(context.get_user_input())
            # Finish as completed, with the title as the final message
            await updater.complete(
                message=new_agent_text_message(
                    responses[-1], context_id=context.context_id,
                ),
            )
        except Exception:
            # Any error ends the task as failed, with a generic message
            await updater.failed(
                message=new_agent_text_message(
                    "The title agent could not complete the request.",
                    context_id=context.context_id,
                ),
            )

    async def cancel(self, context: RequestContext, event_queue: EventQueue) -> None:
        # Refuse with an A2A error; the Azure run can't be stopped here
        raise ServerError(error=TaskNotCancelableError())

# Factory that server.py calls to build the executor
def create_foundry_agent_executor(card: AgentCard) -> FoundryAgentExecutor:
    return FoundryAgentExecutor(card)
```

The task moves from submitted to working, then ends as completed or failed.

This adaptation explicitly rejects cancellation because it cannot cancel an in-flight Azure operation. Emitting a failed task with a “cancelled” message, as the starter does, would not demonstrate that backend work stopped. An implementation that supports cancellation must track and stop that operation too.

The Azure SDK calls above are synchronous even though `run_conversation` is declared `async`. They can block a busy server's event loop; this compact exercise keeps the original SDK style. An application serving concurrent traffic should use async SDK calls or move blocking work off the event loop. Next, [unit 04](04-host-a2a-agent-server.md) makes the executor available over HTTP.
