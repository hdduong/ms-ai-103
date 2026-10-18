# 02 Define an A2A agent

[Define an A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/2-define-a2a-agent?pivots=text)

- A2A is how agents from different platforms call each other. Each agent can use its own model. Auth is part of the protocol. MCP is tools for one model. A2A is agents talking to agents.
- A **skill** is one capability: id, name, description, tags, examples, and input/output modes (text, JSON).
- An **Agent Card** is the discovery document: name, description, version, endpoint, streaming or push, default modes, the skill list, and whether credentials are required.
- A router reads the cards and sends the request to the skill that matches. One agent’s output can be the next agent’s input.
