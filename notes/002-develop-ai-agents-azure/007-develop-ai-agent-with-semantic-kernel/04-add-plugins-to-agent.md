# 04 Add tools to Azure AI agent

[Add tools to Azure AI agent](https://learn.microsoft.com/en-us/training/modules/develop-ai-agent-with-semantic-kernel/4-add-plugins-to-agent?pivots=text)

- Service tools (Foundry runs them): code interpreter, file search, web search, hosted MCP, Azure AI Search, Foundry toolboxes. Some of these are still preview.
- Custom tools are Python functions you pass in. The framework builds the schema from type hints and the docstring, or from `@tool` (name, description, optional Pydantic schema).
- Pass a list. The model picks the tool. You do not route.
- `approval_mode` on `@tool` pauses before a call that spends money, deletes data, or touches secrets.
- One agent can be a tool of another. That is the handoff into the multi-agent module.
- Describe the function and every parameter. Return data the model can read. One job per tool. Return an error string instead of raising, so the model can explain the failure.
