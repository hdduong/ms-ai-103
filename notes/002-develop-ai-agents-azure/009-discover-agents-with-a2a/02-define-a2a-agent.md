# 02 Define an A2A agent

[Define an A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/2-define-a2a-agent?pivots=text)

- A2A standardizes communication between agents, including discovery and task/message exchange. Each agent can use its own backend. MCP standardizes access to tools, resources, and prompts; it is not restricted to one model.
- A2A can describe authentication requirements, but each deployment must configure authentication and authorization. The local exercise does not secure its A2A HTTP endpoints merely by using Azure credentials for backend calls.
- A **skill** is one capability: id, name, description, tags, examples, and input/output modes (text, JSON).
- An **Agent Card** is the discovery document: name, description, version, endpoint, streaming or push, default modes, the skill list, and whether credentials are required.
- A router reads the cards and sends the request to the skill that matches. One agent's output can be the next agent's input.

## Skill and card

Lab: [Connect to remote agents with A2A](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/09-multi-remote-agents-with-a2a.html). Use `a2a-sdk==0.3.26` as described in [06-exercise.md](06-exercise.md). The card's `version` describes this agent application; it is not the SDK/protocol version.

```python
from a2a.types import AgentCapabilities, AgentCard, AgentSkill

host = "localhost"
port = 10007

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
