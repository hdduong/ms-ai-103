# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

- Three parts: the Agent Card, a request handler, and the HTTP app.
- The card is served at `/.well-known/agent-card.json`. An extended card can require auth.
- The handler sends `execute` and `cancel` to the executor. A Task Store tracks tasks, streams, and resubscribe. Even a greeting agent needs one. `InMemoryTaskStore` is lost on restart.
- Python stack in the unit: Starlette plus Uvicorn.

## Lab: `title_agent/server.py`

From the [A2A lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). `agent_card`, `host`, and `port` are the ones from [02](02-define-a2a-agent.md#skill-and-card), defined at the top of this file. The extra `/health` route matters: `run_all.py` waits for it before it starts the client.

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

Without the health route, `uvicorn.run(a2a_app.build(), host=host, port=port)` is enough. After it starts, `http://localhost:10007/.well-known/agent-card.json` returns the card. The outline agent is the same shape on port 10008.
