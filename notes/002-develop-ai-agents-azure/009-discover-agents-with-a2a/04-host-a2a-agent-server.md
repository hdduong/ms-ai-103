# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

- Three parts: the Agent Card, a request handler, and the HTTP app.
- The card is served at `/.well-known/agent-card.json`. An extended card can require auth.
- The handler sends `execute` and `cancel` to the executor. A Task Store tracks tasks, streams, and resubscribe. Even a greeting agent needs one.
- Python stack in the unit: Starlette plus Uvicorn.

## Starlette and Uvicorn

From the lab's `title_agent/server.py`. `agent_card` is the card from unit 02. `build()` turns the A2A app into a Starlette app, and Uvicorn serves it.

```python
import uvicorn
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore

agent_executor = create_foundry_agent_executor(agent_card)
request_handler = DefaultRequestHandler(
    agent_executor=agent_executor,
    task_store=InMemoryTaskStore(),
)
a2a_app = A2AStarletteApplication(
    agent_card=agent_card,
    http_handler=request_handler,
)
uvicorn.run(a2a_app.build(), host=host, port=port)
```

After it starts, `http://{host}:{port}/.well-known/agent-card.json` returns the card.
