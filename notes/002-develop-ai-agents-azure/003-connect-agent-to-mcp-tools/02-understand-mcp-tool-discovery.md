# 02 Understand MCP tool discovery

[Understand MCP tool discovery](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/2-understand-mcp-tool-discovery?pivots=text)

- MCP is a catalog the agent reads at runtime. Add or change a tool on the server and the agent code stays the same.
- Works across models, so you can swap the LLM without rewriting each integration. Auth is one pattern, not a key per API.
- Server hosts tools (`@mcp.tool`). Client lists them. The agent calls them.
- Use it when tools change often or several teams own the APIs.
