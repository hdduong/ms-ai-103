# 03 Implement an agent executor

[Implement an agent executor](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/3-implement-agent-executor?pivots=text)

- `AgentExecutor` is the protocol side of your logic. It does not decide discovery. The card does that.
- `execute` reads `RequestContext`, runs your code, and puts messages, task updates, or artifacts on the `EventQueue`.
- `cancel` stops a running task. A simple agent can refuse it.
- Flow: request comes in, executor calls your helper, the result becomes an event, the server sends the event back.

## Execute and cancel

```python
from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events.event_queue import EventQueue
from a2a.utils import new_agent_text_message, new_task

class TitleExecutor(AgentExecutor):
    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        task = context.current_task or new_task(context.message)
        await event_queue.enqueue_event(task)
        await event_queue.enqueue_event(
            new_agent_text_message("Blob access tiers, in one page", task.context_id, task.id),
        )

    async def cancel(self, context: RequestContext, event_queue: EventQueue) -> None:
        raise RuntimeError("cancel is not supported")
```
