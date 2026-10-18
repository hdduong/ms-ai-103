# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: `always` (default) pauses for a person. `never` runs the tool. If approval is required, the response has `mcp_approval_request`. Reply with `mcp_approval_response` (`approval_request_id` and `approve`).

```python
mcp_tool = MCPTool(
    server_label="github",
    server_url="https://api.githubcopilot.com/mcp/",
    require_approval="always",
)
agent = project_client.agents.create_version(
    name="github-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="Use the GitHub MCP tools when the user asks about a repository.",
        tools=[mcp_tool],
    ),
)
print(agent.name)
```
