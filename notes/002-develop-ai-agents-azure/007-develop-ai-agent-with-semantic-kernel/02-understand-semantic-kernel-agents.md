# 02 Understand Microsoft Agent Framework AI agents

[Understand Microsoft Agent Framework AI agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/2-understand-semantic-kernel-agents?pivots=text)

- Agent Framework replaces Semantic Kernel and AutoGen. Same teams. One `Agent` base class for every provider.
- Pieces: model clients, agent session, context providers (memory), function tools, MCP clients, middleware, graph workflows.
- Every agent can call functions, keep multi-turn history, return structured output, and stream. Hosted tools (code, file search, web) depend on the provider.
- Foundry is the production provider because chat history lives on the service. Azure OpenAI Responses and OpenAI Responses also keep service history. Chat Completions, Claude, Bedrock, GitHub Copilot, and Ollama keep history only in the process.
