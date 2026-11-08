# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: `always` requires an approval response; `never` allows execution without one. These are strings, not booleans. When the response contains `mcp_approval_request`, the client supplies `mcp_approval_response` with `approval_request_id` and `approve`. Your application implements the user decision or an explicit automatic policy.

## Remote MCP tool

The exercise uses the public Microsoft Learn MCP endpoint. Set `require_approval="always"` to request approval when the model chooses a tool. A response can contain an `mcp_approval_request`; the model can also answer without a tool, so this is not a guarantee that the first response always requests approval.

```python
from azure.ai.projects.models import MCPTool, PromptAgentDefinition

mcp_tool = MCPTool(
    server_label="api-specs",
    server_url="https://learn.microsoft.com/api/mcp",
    require_approval="always",
)
agent = project_client.agents.create_version(
    agent_name="MyAgent",
    definition=PromptAgentDefinition(
        model="gpt-5",  # Your deployed model name.
        instructions="Use Microsoft Learn MCP tools for current Azure documentation.",
        tools=[mcp_tool],
    ),
)
```

`project_client` is an initialized `AIProjectClient` in this definition-only fragment. [The complete exercise](05-exercise-mcp-tools.md) includes authentication, a conversation, a human approval/denial loop, execution, and cleanup. The original lab auto-approves requests to its public documentation server; that is a demonstration policy, not a human approval interface.
