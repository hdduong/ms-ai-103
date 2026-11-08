# 02 Understand Microsoft Agent Framework AI agents

[Understand Microsoft Agent Framework AI agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/2-understand-semantic-kernel-agents?pivots=text)

- Microsoft Agent Framework brings together ideas from Semantic Kernel and AutoGen. The current Learn module teaches Agent Framework, although its URL and this folder still say `semantic-kernel`. Its Python examples use the `Agent` abstraction; this is not a claim that every older Semantic Kernel or AutoGen application has migrated.
- Pieces: model clients, an agent session, context providers, function tools, MCP clients, middleware, and graph workflows (sequential, concurrent, group chat, handoff).
- Agents support function tools, multi-turn sessions, structured output, and streaming. Built-in tools such as code execution, file search, and web search depend on the selected provider, model, and service configuration.
- Foundry/Responses clients can use service-managed conversation state. Other clients can maintain history through the framework's session/context providers. Service storage alone does not reconnect a restarted application to its prior conversation: retain and restore the relevant session state. Application-managed history can also be persisted; it is not inherently limited to one process.

The [completed exercise](05-exercise.md) uses `agent-framework-core==1.12.1` and `agent-framework-foundry==1.10.3`. This configuration excerpt uses the same client, followed in the full lesson by the tool and agent:

```python
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential

client = FoundryChatClient(
    project_endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
    model="gpt-5",  # replace with your deployment name
    credential=AzureCliCredential(),
)
print(client)
```
