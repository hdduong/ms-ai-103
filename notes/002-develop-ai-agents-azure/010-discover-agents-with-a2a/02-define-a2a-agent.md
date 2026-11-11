# 02 Define an A2A agent

[Define an A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/2-define-a2a-agent?pivots=text)

A2A lets separately hosted agents discover one another and exchange work. Each can use its own model or framework. MCP connects applications to tools; A2A exposes an agent's capabilities through a protocol endpoint.

- A **skill** describes one capability with an id, name, description, tags, and example requests.
- An **Agent Card** advertises the agent's endpoint, input/output modes, skills, and protocol capabilities.
- A client reads the card before sending a request. Advertising authentication requirements does not configure authentication by itself.

## Lab setup

The [Microsoft Learn A2A exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html) uses a title agent, an outline agent, and a router. Their backend uses `azure.ai.agents.AgentsClient` with **threads and runs**. Keep that SDK generation separate from the Agent Framework and Responses examples in earlier modules.

Use Python **3.13**, a Foundry project, and a deployed model such as the exercise's `gpt-5`. The starter contains the launcher, outline agent, router, and client, so clone it before assembling the code in these lessons:

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
git -C mslearn-ai-agents checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd mslearn-ai-agents/Labfiles/09-build-remote-agents-with-a2a/python
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

On PowerShell, use `.\.venv\Scripts\Activate.ps1`. The [requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/09-build-remote-agents-with-a2a/python/requirements.txt) pin `a2a-sdk==0.3.26` and cap `openai<3`; the remaining packages are not fully locked. Create `.env` in this `python` folder:

```dotenv
PROJECT_ENDPOINT="https://<resource-name>.services.ai.azure.com/api/projects/<project-name>"
MODEL_DEPLOYMENT_NAME="gpt-5"
SERVER_URL="localhost"
ROUTING_AGENT_PORT=10009
OUTLINE_AGENT_PORT=10008
TITLE_AGENT_PORT=10007
```

The following lessons complete the starter by responsibility:

| File | What to do | Lesson |
| --- | --- | --- |
| `title_agent/server.py` | Replace with the card block below, then append the HTTP block | [02](#skill-and-card) and [04](04-host-a2a-agent-server.md#lab-title_agentserverpy) |
| `title_agent/agent.py` | Replace with the completed model backend | [03](03-implement-agent-executor.md#lab-the-title-agents-logic) |
| `title_agent/agent_executor.py` | Replace with the completed protocol executor | [03](03-implement-agent-executor.md#a2a-task-events) |
| `routing_agent/agent.py` | Replace the entire `send_message` method and repair its polling delay | [05](05-connect-to-a2a-agent.md#lab-the-routers-send_message) |
| Outline agent, launcher, terminal client | Keep the provided implementation | [05](05-connect-to-a2a-agent.md#run-all-three) |

## Skill and card

Start `title_agent/server.py` with this block. It loads local configuration and defines what a remote client can discover. Append the HTTP application from [unit 04](04-host-a2a-agent-server.md#lab-title_agentserverpy) to finish the file.

```python
import os
from a2a.types import AgentCapabilities, AgentCard, AgentSkill
from dotenv import load_dotenv

load_dotenv()
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
```

The skill describes the behavior the router should select; it is not a Python function registration. `url` points to the server that will accept requests. `version="1.0.0"` is this agent application's version, not the installed A2A SDK or protocol version. The card alone performs no model inference; execution belongs to the [backend and executor](03-implement-agent-executor.md).
