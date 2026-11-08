# 05 Connect to your A2A agent

[Connect to your A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/5-connect-to-a2a-agent?pivots=text)

- The client needs the server base URL, then fetches the Agent Card, then sends messages.
- Non-streaming returns one response envelope; a task in it may already be complete or may still be running. Streaming exposes incremental message/task events when supported.
- A message has a role (`user`) and content, and a unique id.
- A successful result is a direct message or a task. Inspect a task's status before polling or following a stream; do not treat a completed task as ongoing work.

## One message

The routing agent in the exercise keeps one client wrapper per discovered card, in `self.remote_agent_connections`. This helper builds a payload and awaits the wrapper's `send_message`; invoke it from an async caller. See [06-exercise.md](06-exercise.md) for the missing starter blocks and the router's Task-only result checks.

```python
import uuid
from typing import Any
from a2a.types import MessageSendParams, SendMessageRequest, SendMessageResponse

async def send_task(client, task: str) -> SendMessageResponse:
    message_id = str(uuid.uuid4())
    payload: dict[str, Any] = {
        "message": {
            "role": "user",
            "parts": [{"kind": "text", "text": task}],
            "messageId": message_id,
        },
    }
    message_request = SendMessageRequest(
        id=message_id,
        params=MessageSendParams.model_validate(payload),
    )
    return await client.send_message(message_request=message_request)

async def ask_title(client):
    send_response = await send_task(
        client, "Create a title for an article about React programming."
    )
    print(send_response)
```

`await` obtains the response rather than an unexecuted coroutine. A generic client should handle both protocol success and error envelopes, and both successful result types (`Message` and `Task`).
