# 03 Implement an agent executor

[Implement an agent executor](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/3-implement-agent-executor?pivots=text)

- `AgentExecutor` is the protocol side of your logic. It does not decide discovery. The card does that.
- `execute` reads `RequestContext`, runs your code, and puts messages, task updates, or artifacts on the `EventQueue`.
- `cancel` handles a cancellation request. It must actually stop the underlying operation when supported, or return a protocol error. Emitting a cancelled/failed status alone does not stop a backend Azure run.
- Flow: request comes in, executor calls your helper, the result becomes an event, the server sends the event back.

## Execute and cancel

Lab: [Connect to remote agents with A2A](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). The title agent's executor reports progress with a `TaskUpdater`, runs the Foundry agent, then completes the task with the last reply. `self.agent` is the lab's title agent wrapper. This is a constructor-injection teaching excerpt, not the exact starter file. [06-exercise.md](06-exercise.md) supplies the complete integrated executor and setup.

```python
from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events.event_queue import EventQueue
from a2a.server.tasks import TaskUpdater
from a2a.types import TaskNotCancelableError, TaskState
from a2a.utils import new_agent_text_message, new_task
from a2a.utils.errors import ServerError

class TitleAgentExecutor(AgentExecutor):
    def __init__(self, agent):
        self.agent = agent

    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        task = context.current_task or new_task(context.message)
        await event_queue.enqueue_event(task)
        updater = TaskUpdater(event_queue, task.id, task.context_id)
        await updater.update_status(
            TaskState.working,
            message=new_agent_text_message("Title Agent is processing your request...", context_id=task.context_id),
        )
        responses = await self.agent.run_conversation(context.get_user_input())
        final_message = responses[-1] if responses else "Task completed."
        await updater.complete(
            message=new_agent_text_message(final_message, context_id=task.context_id),
        )

    async def cancel(self, context: RequestContext, event_queue: EventQueue) -> None:
        raise ServerError(error=TaskNotCancelableError())
```
