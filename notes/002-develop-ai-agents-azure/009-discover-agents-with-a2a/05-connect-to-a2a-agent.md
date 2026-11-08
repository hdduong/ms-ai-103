# 05 Connect to your A2A agent

[Connect to your A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/5-connect-to-a2a-agent?pivots=text)

- The client needs the server base URL, then fetches the Agent Card, then sends messages.
- Non-streaming waits for one reply. Streaming returns chunks. Use streaming for a long task.
- A message has a role (`user`) and content, and a unique id.
- The reply is either a direct message or a task. A task means you poll or follow the stream. Advanced agents run more than one task at a time.

## One message

The routing agent in the exercise keeps one A2A client per agent card it discovered, in `self.remote_agent_connections`. It builds this payload and awaits `send_message`.

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

client = self.remote_agent_connections[agent_name]
send_response = await send_task(client, "Create a title for an article about React programming.")
print(send_response)
```

`send_message` waits for the full reply. Leave out the `await` and the routing run never gets the title back.
