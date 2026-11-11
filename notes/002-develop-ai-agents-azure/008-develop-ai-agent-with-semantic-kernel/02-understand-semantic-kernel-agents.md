# 02 Understand Microsoft Agent Framework AI agents

[Understand Microsoft Agent Framework AI agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/2-understand-semantic-kernel-agents?pivots=text)

- Microsoft Agent Framework is the next generation of Semantic Kernel and AutoGen, built by the same teams. The module URL still says semantic-kernel; these examples use `Agent` from Agent Framework.
- The **chat client** connects to the model provider. The **agent** combines that client with instructions and tools. A **session** carries conversation state between runs.
- Context providers supply history or other context; middleware handles cross-cutting behavior. Workflows coordinate agents through a graph.
- Functions, streaming, structured output, and hosted tools depend on the selected provider and model. Passing a tool does not make an unsupported model able to use it.

## Where conversation state lives

The Learn provider table distinguishes service-managed history from history managed by your application:

| Provider | Service-managed chat history |
| --- | --- |
| Foundry Agent Service | Supported |
| Azure OpenAI Responses, OpenAI Responses | Supported |
| Azure OpenAI Chat Completion, OpenAI Chat Completion | Application manages messages |
| Anthropic Claude, Amazon Bedrock, GitHub Copilot, Ollama | Application manages messages |

The [Agent Framework session documentation](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/conversations/session) describes how to carry state across calls. A new `agent.run(...)` call does not automatically find a previous conversation. Pass the same `AgentSession` to connect turns. To continue after a restart, persist and restore the session, including its service identifier where applicable. Application-managed history can also be persisted; it is lost only if you keep it solely in process memory.

With the [packages in unit 03](03-create-azure-ai-agent.md#one-run), this local example shows the state container without calling a model:

```python
from agent_framework import AgentSession

session = AgentSession()
saved_state = session.to_dict()
restored_session = AgentSession.from_dict(saved_state)
assert restored_session.session_id == session.session_id
```

`to_dict()` produces data your application can store; it does not write a file or database record. The empty session above has no history yet. In the expense scenario, [unit 03](03-create-azure-ai-agent.md) makes one independent request, and [unit 04](04-add-plugins-to-agent.md) adds the function that acts on its result.
