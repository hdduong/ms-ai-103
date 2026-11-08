# 02 Understand Microsoft Agent Framework AI agents

[Understand Microsoft Agent Framework AI agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/2-understand-semantic-kernel-agents?pivots=text)

- Microsoft Agent Framework replaces Semantic Kernel and AutoGen. The module URL still says semantic-kernel. Every agent is an `Agent`.
- Pieces: model clients, an agent session, context providers, function tools, MCP clients, middleware, and graph workflows (sequential, concurrent, group chat, handoff).
- Every agent can call functions, keep a multi-turn session, return structured output, stream, and use provider tools such as code execution, file search, and web search.
- Foundry is the production provider because chat history stays in the service. Responses providers also keep service history. Chat Completions, Claude, Bedrock, Copilot, and Ollama keep history only in the process.

The module exercise builds the client the same way, then wraps a tool and runs the agent. The client from that lab:

```python
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential

client = FoundryChatClient(
    project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
    model="gpt-4.1",  # deployment name
    credential=AzureCliCredential(),
)
print(client)
```
