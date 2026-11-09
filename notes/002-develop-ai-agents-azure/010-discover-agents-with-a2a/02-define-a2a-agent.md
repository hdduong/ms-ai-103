# 02 Define an A2A agent

[Define an A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/2-define-a2a-agent?pivots=text)

- A2A is how agents from different platforms call each other. Each agent can use its own model. Auth is part of the protocol, but each deployment still has to turn it on. MCP connects a model to tools. A2A is agents talking to agents.
- A **skill** is one capability: id, name, description, tags, examples, and input/output modes (text, JSON).
- An **Agent Card** is the discovery document: name, description, version, endpoint, streaming or push, default modes, the skill list, and whether credentials are required.
- A router reads the cards and sends the request to the skill that matches. One agent's output can be the next agent's input.

## Lab setup

Lab: [Connect to remote agents with A2A](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). A title agent and an outline agent each publish a card. A routing agent discovers both and delegates. The backend is the Azure AI Agents SDK (threads and runs), not Agent Framework.

```bash
pip install python-dotenv httpx azure-identity uvicorn starlette sse-starlette fastapi azure-ai-projects azure-ai-agents a2a-sdk==0.3.26 "openai<3"
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>"
MODEL_DEPLOYMENT_NAME="gpt-5"
SERVER_URL="localhost"
ROUTING_AGENT_PORT=10009
OUTLINE_AGENT_PORT=10008
TITLE_AGENT_PORT=10007
```

## Skill and card

The title agent's card, from `title_agent/server.py`. `version` is the agent's own version, not the SDK's.

```python
import os
from a2a.types import AgentCapabilities, AgentCard, AgentSkill

host = os.environ["SERVER_URL"]
port = int(os.environ["TITLE_AGENT_PORT"])

skills = [
    AgentSkill(
        id="generate_blog_title",
        name="Generate Blog Title",
        description="Generates a blog title based on a topic",
        tags=["title"],
        examples=["Can you give me a title for this article?"],
    ),
]
agent_card = AgentCard(
    name="Microsoft Foundry Title Agent",
    description="An intelligent title generator agent powered by Foundry.",
    url=f"http://{host}:{port}/",
    version="1.0.0",
    default_input_modes=["text"],
    default_output_modes=["text"],
    capabilities=AgentCapabilities(),
    skills=skills,
)
print(agent_card.name, skills[0].id)
```
