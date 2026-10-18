# 04 Host an A2A server

[Host an A2A server](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/4-host-a2a-agent-server?pivots=text)

- Three parts: the Agent Card, a request handler, and the HTTP app.
- The card is served at `/.well-known/agent-card.json`. An extended card can require auth.
- The handler sends `execute` and `cancel` to the executor. A Task Store tracks tasks, streams, and resubscribe. Even a greeting agent needs one.
- Python stack in the unit: Starlette plus Uvicorn. Start the server and it listens for A2A requests.
