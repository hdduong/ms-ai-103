# 05 Connect to your A2A agent

[Connect to your A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/5-connect-to-a2a-agent?pivots=text)

A client starts with an agent's base URL, retrieves its card, then sends a message matching an advertised skill. A message carries a role, content parts, and an id. A reply can be a direct `Message` or a `Task` describing work and its current state.

A task may already be completed when a non-streaming request returns. If it is still running, inspect its state and use polling or streaming as appropriate; do not assume that every task requires another poll.

## Lab: the router's `send_message`

The [provided router](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/routing_agent/agent.py) discovers the title and outline cards with `A2ACardResolver`. It stores a connection under each card's name and advertises a `send_message` function to its own routing model. That function is where one model-selected task becomes a remote A2A request.

In `routing_agent/agent.py`, replace the **entire `RoutingAgent.send_message` method**, from its `async def` line through its final return, with the block below. Indent it inside the existing class. This is a replacement method, not a standalone program; the starter already imports `uuid`, `Any`, `MessageSendParams`, `SendMessageRequest`, `SendMessageSuccessResponse`, and `Task`.

```python
async def send_message(self, agent_name: str, task: str):
    """Send a task to a previously discovered remote agent."""
    # agent_name comes from the model; accept only discovered agents
    if agent_name not in self.remote_agent_connections:
        raise ValueError(f"Agent {agent_name} not found")
    # Connection created for this agent's card during discovery
    client = self.remote_agent_connections[agent_name]
    if client is None:
        raise ValueError(f"Client not available for {agent_name}")

    # Every A2A message needs a unique messageId
    message_id = str(uuid.uuid4())
    # A2A message: the task as one text part, sent with the user role
    payload: dict[str, Any] = {
        "message": {
            "role": "user",
            "parts": [{"kind": "text", "text": task}],
            "messageId": message_id,
        },
    }
    # Wrap it in a JSON-RPC message/send request; validates the dict
    message_request = SendMessageRequest(
        id=message_id,
        params=MessageSendParams.model_validate(payload),
    )
    # POST to the remote agent over HTTP and wait for the reply
    send_response = await client.send_message(message_request=message_request)
    # The reply is a JSON-RPC success or error; fail on errors
    if not isinstance(send_response.root, SendMessageSuccessResponse):
        raise RuntimeError("Remote agent returned an A2A error response")
    # A result can be a Task or a direct Message; this lab expects a Task
    if not isinstance(send_response.root.result, Task):
        raise RuntimeError("This lab router expects an A2A Task result")
    # Return the Task; the starter submits it as the tool output
    return send_response.root.result
```

The `Task` check matches the two executors used here. A general-purpose A2A client must also handle direct `Message` results. A2A task/context ids remain distinct from the Azure thread/run ids used by each backend.

### Resume the routing model with the result

The starter already handles the routing model's `requires_action` state: it reads the `send_message` arguments, awaits this method, serializes its result, and submits a tool output with the original `tool_call_id`. That id associates the returned title or outline with the model's pending function call. Removing `await` would return a coroutine instead of the remote result.

In the same file, replace `time.sleep(1)` inside `process_user_message` with `await asyncio.sleep(1)`. `asyncio` is already imported. This replacement belongs inside the existing async method, not at file scope. It allows other coroutines to run while polling waits. The remaining Azure SDK calls are synchronous, as explained in [unit 03](03-implement-agent-executor.md).

The file also ends with an unused `_get_initialized_routing_agent_sync` helper and `routing_agent = _get_initialized_routing_agent_sync()`. Remove that helper and assignment; the [router server's lifespan](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/routing_agent/server.py) already awaits `RoutingAgent.create(...)` during startup. The unused module-level call otherwise creates an unawaited coroutine.

## Run all three

After completing the [file map in unit 02](02-define-a2a-agent.md#lab-setup), remain in the starter's `Labfiles/09-build-remote-agents-with-a2a/python` directory and run:

```bash
# Start the three servers (waiting on each /health), then the client
python run_all.py
```

| Component | Port | Interface |
| --- | --- | --- |
| Title agent | 10007 | A2A card and task endpoint |
| Outline agent | 10008 | A2A card and task endpoint |
| Routing application | 10009 | Plain HTTP `/message` for the terminal client |

Ask the terminal client:

```text
Create a title and outline for an article about React programming.
```

The router discovers the agents, delegates work using their card names, and incorporates their returned tasks into its final response. Check that both requested outputs appear and that remote tasks finish as completed or failed; generated wording and the model's delegation order may vary.

Stopping `run_all.py` ends the local servers. It leaves Azure agent definitions, the router thread, and the supplied outline agent's threads; remove those disposable resources after use. The adapted title backend deletes its own per-request thread. The supplied outline executor also labels cancellation as a failed task without stopping backend work, so it should not be treated as a complete cancellation implementation.
