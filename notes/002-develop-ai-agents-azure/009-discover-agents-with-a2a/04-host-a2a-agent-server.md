# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

- Three parts: the Agent Card, a request handler, and the HTTP app.
- The card is served at `/.well-known/agent-card.json`. An extended card can require auth.
- `DefaultRequestHandler` links protocol requests to the executor and uses a Task Store for task state. Event queues and request handling support streaming/resubscription. `InMemoryTaskStore` is process-local and loses task records at restart.
- Python stack in the unit: Starlette plus Uvicorn.

## Starlette and Uvicorn

Condensed from the lab's `title_agent/server.py`. This helper expects the card and executor to be supplied. `build()` turns the A2A app into a Starlette app, and Uvicorn serves it. The [full exercise](06-exercise.md) retains `/health`, which the supplied launcher needs.

```python
import uvicorn
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore

def serve_agent(agent_card, agent_executor, host="localhost", port=10007):
    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor,
        task_store=InMemoryTaskStore(),
    )
    a2a_app = A2AStarletteApplication(
        agent_card=agent_card,
        http_handler=request_handler,
    )
    uvicorn.run(a2a_app.build(), host=host, port=int(port))
```

After it starts, `http://{host}:{port}/.well-known/agent-card.json` returns the card.
