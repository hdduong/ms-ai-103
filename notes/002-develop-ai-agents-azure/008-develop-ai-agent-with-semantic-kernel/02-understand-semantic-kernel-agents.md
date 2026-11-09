# 02 Understand Microsoft Agent Framework AI agents

[Understand Microsoft Agent Framework AI agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/2-understand-semantic-kernel-agents?pivots=text)

- Microsoft Agent Framework is the next generation of Semantic Kernel and AutoGen, built by the same teams. The module URL still says semantic-kernel. Every agent is an `Agent`.
- Pieces: model clients, an agent session, context providers, function tools, MCP clients, middleware, and graph workflows (sequential, concurrent, group chat, handoff).
- Every agent can call functions, keep a multi-turn session, return structured output, stream, and use provider tools such as code execution, file search, and web search.
- Foundry is the recommended production provider because chat history stays in the service.

| Provider | Service chat history |
| --- | --- |
| Foundry Agent Service | Yes |
| Azure OpenAI Responses, OpenAI Responses | Yes |
| Azure OpenAI Chat Completion, OpenAI Chat Completion | No |
| Anthropic Claude, Amazon Bedrock, GitHub Copilot, Ollama | No |

"No" means the history lives in your process and is gone when it stops.

The client from the lab:

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
