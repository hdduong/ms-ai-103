# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

- Three parts: the Agent Card, a request handler, and the HTTP app.
- The card is served at `/.well-known/agent-card.json`. An extended card can require auth.
- The handler sends `execute` and `cancel` to the executor. A Task Store tracks tasks, streams, and resubscribe. Even a greeting agent needs one.
- Python stack in the unit: Starlette plus Uvicorn.

## Starlette and Uvicorn

```python
import uvicorn
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore

handler = DefaultRequestHandler(
    agent_executor=TitleExecutor(),
    task_store=InMemoryTaskStore(),
)
server = A2AStarletteApplication(agent_card=card, http_handler=handler)
uvicorn.run(server.build(), host="0.0.0.0", port=9000)
```

`card` is the Agent Card from unit 02. `TitleExecutor` is the executor from unit 03.
