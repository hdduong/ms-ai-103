# 06 Exercise — Connect remote agents with A2A

Reviewed **2026-11-07** against [Microsoft Learn's A2A module](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/) and the [official exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). The source snapshot used below is [`MicrosoftLearning/mslearn-ai-agents@4d6a326`](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python).

The exercise builds a local routing application that discovers two remote agents: a title writer and an outline writer. Each remote agent exposes an Agent Card and an A2A endpoint. The routing agent decides which remote agent to call, awaits its result, and submits that result back to its own Foundry run.

## Keep the two SDK layers separate

This exercise currently uses `a2a-sdk==0.3.26` for communication and **`azure.ai.agents.AgentsClient` with agents, threads, messages, and runs** for the model backend. Modules 007 and 008 instead use Microsoft Agent Framework's `FoundryChatClient`. Their constructors, session types, and response types are not interchangeable. The A2A protocol does not require either backend; it only defines the interface exposed to other agents.

The upstream folder is a **starter**, with deliberate gaps in the title agent and router. Complete those gaps before running it. The following sections provide completed teaching versions of the title files and the router's missing request method. They preserve the official exercise architecture and add explicit error and cancellation behavior. They are not a claim that an untouched starter runs.

## 1. Project, environment, and dependencies

Follow the official exercise to create/select a Foundry project, deploy the model, and obtain the project endpoint. Its tested environment is **Python 3.13.12**; the instructions state that Python 3.14 is not supported. Use a separate virtual environment from the Agent Framework lessons.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/09-build-remote-agents-with-a2a/python
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

The [snapshot's requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/requirements.txt) are:

```text
python-dotenv
httpx
azure-identity
uvicorn
starlette
sse-starlette
fastapi
azure-ai-projects
azure-ai-agents
a2a-sdk==0.3.26
openai<3
```

Only A2A is pinned to an exact release in this lab; these requirements are not a complete dependency lock. The upstream comment caps OpenAI below 3 because its HTTP client change is incompatible with the lab's Azure project dependency. The API examples here were also checked against the published `azure-ai-agents==1.1.0` source, but no Azure request was executed during this review. Keep a successful environment's resolved versions when reproducing a course run.

Use the names in the lab's `.env`:

```dotenv
PROJECT_ENDPOINT="https://YOUR-RESOURCE.services.ai.azure.com/api/projects/YOUR-PROJECT"
MODEL_DEPLOYMENT_NAME="gpt-5"
SERVER_URL="localhost"
ROUTING_AGENT_PORT=10009
OUTLINE_AGENT_PORT=10008
TITLE_AGENT_PORT=10007
```

The model value is your **deployment name**. Azure credentials authenticate the backend SDK; they do not automatically authenticate the local HTTP/A2A endpoints. These lab servers are local demonstrations and do not configure A2A endpoint authentication.

## 2. Complete the title agent backend

Replace `title_agent/agent.py` with this completed adaptation of the [official starter](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/title_agent/agent.py) and [exercise instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/09-multi-remote-agents-with-a2a.md). The surrounding server loads `.env` before this wrapper is created.

```python
import os

from azure.ai.agents import AgentsClient
from azure.ai.agents.models import Agent, ListSortOrder, MessageRole
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
        self.agent: Agent | None = None

    async def create_agent(self) -> Agent:
        if self.agent is None:
            self.agent = self.client.create_agent(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],
                name="title-agent",
                instructions=(
                    "Generate one clear, catchy blog post title for the user's "
                    "topic. Return the title only."
                ),
            )
        return self.agent

    async def run_conversation(self, user_message: str) -> list[str]:
        agent = await self.create_agent()
        thread = self.client.threads.create()
        try:
            self.client.messages.create(
                thread_id=thread.id,
                role=MessageRole.USER,
                content=user_message,
            )
            run = self.client.runs.create_and_process(
                thread_id=thread.id,
                agent_id=agent.id,
            )
            if run.status != "completed":
                raise RuntimeError(f"Title run ended with status {run.status}")

            messages = self.client.messages.list(
                thread_id=thread.id,
                order=ListSortOrder.DESCENDING,
            )
            for message in messages:
                if message.role == MessageRole.AGENT and message.text_messages:
                    return [part.text.value for part in message.text_messages]
            raise RuntimeError("Title run completed without a text response")
        finally:
            self.client.threads.delete(thread.id)


async def create_foundry_title_agent() -> TitleAgent:
    wrapper = TitleAgent()
    await wrapper.create_agent()
    return wrapper
```

The adaptation raises a failure when no completed text response exists, so the A2A executor can report a failed task. It also deletes the per-request title thread in `finally`; the upstream starter does not include that cleanup. Each request creates a new title thread, so this wrapper does not preserve a conversation between title requests. The cached **agent definition** remains in the project until explicitly deleted.

The lab uses the synchronous Azure SDK inside `async` methods. Declaring a function `async` does not make those SDK calls nonblocking. This is sufficient to study the exercise's request flow, but concurrent service workloads need the asynchronous Azure client or careful offloading of blocking work.

## 3. Bridge the backend to A2A task events

Replace `title_agent/agent_executor.py` with this shortened, complete adaptation of the [official executor](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/title_agent/agent_executor.py). `RequestContext.get_user_input()` extracts the text instead of assuming the first message part is a text part.

```python
from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events.event_queue import EventQueue
from a2a.server.tasks import TaskUpdater
from a2a.types import AgentCard, TaskNotCancelableError, TaskState
from a2a.utils import new_agent_text_message
from a2a.utils.errors import ServerError

from title_agent.agent import create_foundry_title_agent


class FoundryAgentExecutor(AgentExecutor):
    def __init__(self, card: AgentCard):
        self._card = card
        self._foundry_agent = None

    async def execute(
        self, context: RequestContext, event_queue: EventQueue
    ) -> None:
        updater = TaskUpdater(event_queue, context.task_id, context.context_id)
        await updater.submit()
        await updater.start_work()
        try:
            if self._foundry_agent is None:
                self._foundry_agent = await create_foundry_title_agent()
            await updater.update_status(
                TaskState.working,
                message=new_agent_text_message(
                    "Generating a title...", context_id=context.context_id
                ),
            )
            responses = await self._foundry_agent.run_conversation(
                context.get_user_input()
            )
            await updater.complete(
                message=new_agent_text_message(
                    responses[-1], context_id=context.context_id
                ),
            )
        except Exception:
            await updater.failed(
                message=new_agent_text_message(
                    "The title agent could not complete the request.",
                    context_id=context.context_id,
                ),
            )

    async def cancel(
        self, context: RequestContext, event_queue: EventQueue
    ) -> None:
        raise ServerError(error=TaskNotCancelableError())


def create_foundry_agent_executor(card: AgentCard) -> FoundryAgentExecutor:
    return FoundryAgentExecutor(card)
```

`submit → start_work → complete/failed` describes task state. It does not transfer the underlying Azure thread to the caller. This teaching version explicitly rejects cancellation because it has no implementation that cancels the Azure run. The upstream starter calls `updater.failed()` with a “cancelled” message; that is a failed task status, not proof the Azure work stopped. Implementing real cancellation requires tracking and cancelling the backend operation as well as emitting the appropriate A2A state.

## 4. Publish the card and keep the health route

Replace `title_agent/server.py` with this completed adaptation of the [official server](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/title_agent/server.py). The `/health` route matters because `run_all.py` waits for it before starting the client.

```python
import os

import uvicorn
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore
from a2a.types import AgentCapabilities, AgentCard, AgentSkill
from dotenv import load_dotenv
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.routing import Route

from title_agent.agent_executor import create_foundry_agent_executor

load_dotenv()
host = os.environ["SERVER_URL"]
port = int(os.environ["TITLE_AGENT_PORT"])

skill = AgentSkill(
    id="generate_blog_title",
    name="Generate Blog Title",
    description="Generates a blog post title for a topic.",
    tags=["title", "blog"],
    examples=["Create a title for an article about React programming."],
)
card = AgentCard(
    name="Microsoft Foundry Title Agent",
    description="Generates blog titles using a Foundry model.",
    url=f"http://{host}:{port}/",
    version="1.0.0",
    default_input_modes=["text"],
    default_output_modes=["text"],
    capabilities=AgentCapabilities(),
    skills=[skill],
)
handler = DefaultRequestHandler(
    agent_executor=create_foundry_agent_executor(card),
    task_store=InMemoryTaskStore(),
)
a2a_app = A2AStarletteApplication(agent_card=card, http_handler=handler)


async def health_check(request):
    return PlainTextResponse("Title agent is running")


routes = a2a_app.routes()
routes.append(Route("/health", endpoint=health_check, methods=["GET"]))
app = Starlette(routes=routes)

if __name__ == "__main__":
    uvicorn.run(app, host=host, port=port)
```

The card appears at `http://localhost:10007/.well-known/agent-card.json`. Its `version="1.0.0"` is the **agent's application version**, not the installed A2A SDK version. `InMemoryTaskStore` loses task records when the process restarts; it is not durable task storage. The supplied outline agent has the same server/executor/backend separation.

## 5. Complete the router's awaited remote call

The [provided router](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/routing_agent/agent.py) retrieves each card through `A2ACardResolver`, stores a `RemoteAgentConnections` wrapper by card name, and exposes a `send_message` function tool to the routing model.

Replace the **entire existing `RoutingAgent.send_message` method**, from its `async def` line through `return send_response.root.result`, with the method below, indented inside the `RoutingAgent` class. This is a complete replacement method for the upstream class, not a standalone program. The file already imports `uuid`, `Any`, `MessageSendParams`, `SendMessageRequest`, `SendMessageSuccessResponse`, and `Task`.

```python
async def send_message(self, agent_name: str, task: str):
    """Send a task to a previously discovered remote agent."""
    if agent_name not in self.remote_agent_connections:
        raise ValueError(f"Agent {agent_name} not found")
    client = self.remote_agent_connections[agent_name]
    if client is None:
        raise ValueError(f"Client not available for {agent_name}")

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
        raise RuntimeError("Remote agent returned an A2A error response")
    if not isinstance(send_response.root.result, Task):
        raise RuntimeError("This lab router expects an A2A Task result")
    return send_response.root.result
```

A2A allows either a direct `Message` or a `Task` response. The starter's checks intentionally accept only `Task` results, which matches the title and outline executors in this exercise. Do not assume that check works for every A2A agent. Likewise, receiving a task does not necessarily mean more polling is needed: inspect its state because a non-streaming request can return an already completed task.

While completing the router, make these small local repairs:

- Add `import asyncio` and replace `time.sleep(1)` inside `process_user_message` with `await asyncio.sleep(1)`. Otherwise the polling delay blocks the local event loop. Azure SDK calls in the same method remain synchronous, as noted above.
- Use the documented spelling `MessageRole.USER` for the user message. The inspected `azure-ai-agents==1.1.0` enum uses Azure Core's case-insensitive enum metaclass, so the starter's `MessageRole.User` is not, by itself, evidence of a runtime failure in that release.

The router's `requires_action` loop must **await** `self.send_message(...)`, serialize its result, and submit the tool output with the original `tool_call_id`. Removing `await` produces a coroutine instead of the remote result.

## 6. Run and inspect

After completing the files and router method:

```bash
python run_all.py
```

The [launcher](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/run_all.py) starts three Uvicorn processes, checks health, and runs the terminal client. Ask for a blog title, an outline, and then a request that needs both. The exact generated text and delegation order can vary.

| Component | Local port | Interface |
| --- | --- | --- |
| Title agent | 10007 | A2A Agent Card and task endpoint |
| Outline agent | 10008 | A2A Agent Card and task endpoint |
| Routing application | 10009 | The lab client's ordinary HTTP `/message` endpoint |

The routing application's user-facing endpoint is a FastAPI route; the remote title and outline calls are the A2A part. You can inspect their cards without sending a model request, for example with `curl http://localhost:10007/.well-known/agent-card.json`.

Check that the cards expose the expected names and URLs, requests end with `completed` or `failed`, and the final answer includes the requested remote output. If startup fails, read the corresponding server error: a missing model deployment, project permissions, an unfilled starter section, and a dependency import error are different problems. This review checked source and Python syntax; it did not execute the three-server workflow against Azure.

## Cleanup and what to remember

Exit the terminal client using its displayed quit command or stop the launcher. Its `finally` block terminates the local subprocesses. It does **not** delete Azure agent definitions, the router's thread, or the outline agent's threads. Remove the agents and remaining threads you created, or delete the dedicated exercise resource group if it contains only disposable lab resources. The adapted title wrapper above already deletes its own per-request threads.

Remember the responsibilities: the **card advertises**, the **request handler routes protocol operations**, the **executor performs work and emits task events**, and the **backend agent calls the model**. An A2A message ID, task ID, context ID, Azure thread ID, and Azure run ID serve different purposes.

Code in this lesson is adapted from Microsoft Learn's MIT-licensed exercise, with the corrections called out above. See [third-party notices](../../../THIRD_PARTY_NOTICES.md).
