# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

Hosting connects the pieces built in earlier lessons. The Agent Card advertises capabilities, a request handler dispatches protocol operations to the executor, and an HTTP application exposes the routes.

- The card is available at `/.well-known/agent-card.json`.
- The handler uses a task store to track task state. `InMemoryTaskStore` is suitable for this local example and loses its records when the process restarts.
- An extended card can require authentication. The local exercise below has no authentication middleware and binds to localhost.

## Lab: `title_agent/server.py`

Append this block **after the skill/card block from [unit 02](02-define-a2a-agent.md#skill-and-card)** in `title_agent/server.py`; together they replace the entire starter file. It uses the executor completed in [unit 03](03-implement-agent-executor.md#a2a-task-events).

```python
import uvicorn
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.routing import Route
from title_agent.agent_executor import create_foundry_agent_executor

request_handler = DefaultRequestHandler(
    agent_executor=create_foundry_agent_executor(agent_card),
    task_store=InMemoryTaskStore(),
)
a2a_app = A2AStarletteApplication(
    agent_card=agent_card,
    http_handler=request_handler,
)

async def health_check(request):
    return PlainTextResponse("Title agent is running")

routes = a2a_app.routes()
routes.append(Route("/health", endpoint=health_check, methods=["GET"]))
app = Starlette(routes=routes)

if __name__ == "__main__":
    uvicorn.run(app, host=host, port=port)
```

`DefaultRequestHandler` connects incoming A2A operations to `FoundryAgentExecutor`. `A2AStarletteApplication` creates the protocol and card routes. The extra `/health` route is part of the [exercise launcher](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/run_all.py): it waits for each server to report readiness before continuing. Omitting it can make an otherwise working A2A server fail the launcher's readiness check.

From the exercise's `python` folder, test only the title server with:

```bash
python -m uvicorn title_agent.server:app --host localhost --port 10007
```

In another terminal:

```bash
curl http://localhost:10007/health
curl http://localhost:10007/.well-known/agent-card.json
```

Expect the health text and a card containing `generate_blog_title` and the localhost URL. These requests do not call a model: the backend is created lazily when the executor receives work. Stop this test server before [unit 05's launcher](05-connect-to-a2a-agent.md#run-all-three) starts the same port.

The provided outline agent has the same server/executor/backend separation on port 10008. Reading its card shows how two different capabilities can expose the same protocol shape.
