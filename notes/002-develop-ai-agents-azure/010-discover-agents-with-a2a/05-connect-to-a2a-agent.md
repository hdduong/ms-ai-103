# 05 Connect to your A2A agent

[Connect to your A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/5-connect-to-a2a-agent?pivots=text)

- The client needs the server base URL, then fetches the Agent Card, then sends messages.
- Non-streaming waits for one reply. Streaming returns chunks. Use streaming for a long task.
- A message has a role (`user`) and content, and a unique id.
- The reply is either a direct message or a task. A task means you poll or follow the stream, unless its state is already `completed`. Advanced agents run more than one task at a time.

## Lab: the router's `send_message`

From the [A2A lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). `routing_agent/agent.py` resolves each card with `A2ACardResolver` and keeps one client per card name in `self.remote_agent_connections`. The routing model calls this method as a function tool. It goes inside the `RoutingAgent` class:

```python
async def send_message(self, agent_name: str, task: str):
    """Send a task to a remote agent and return its Task."""
    if agent_name not in self.remote_agent_connections:
        raise ValueError(f"Agent {agent_name} not found")
    client = self.remote_agent_connections[agent_name]

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
    send_response = await client.send_message(message_request=message_request)
    if not isinstance(send_response.root, SendMessageSuccessResponse):
        raise RuntimeError("Remote agent returned an error")
    if not isinstance(send_response.root.result, Task):
        raise RuntimeError("Expected a Task result")
    return send_response.root.result
```

- The router's `requires_action` loop must `await self.send_message(...)` and submit the result with the original `tool_call_id`. Leave out the `await` and the run gets a coroutine, not the title.
- The `Task` check fits these two agents. A general client also accepts a direct `Message`.

## Run all three

```bash
python run_all.py
```

| Component | Port | Interface |
| --- | --- | --- |
| Title agent | 10007 | A2A card and task endpoint |
| Outline agent | 10008 | A2A card and task endpoint |
| Routing agent | 10009 | Plain HTTP `/message` for the lab client |

```text
Create a title and outline for an article about React programming.
```

`curl http://localhost:10007/.well-known/agent-card.json` shows a card without calling a model. Stopping `run_all.py` ends the servers but leaves the Azure agents and threads; delete them in the portal.
