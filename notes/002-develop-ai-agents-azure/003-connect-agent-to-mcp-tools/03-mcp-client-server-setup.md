# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

- Server: `FastMCP("server-name")`. Type hints and docstrings become the tool schema. Served over HTTP.
- Client: open a session, `session.list_tools()`, wrap each tool in an async function that calls `session.call_tool(name, args)`, bundle them as a `FunctionTool`, register that on the agent.
- This path is the manual client. The next unit is the Foundry `MCPTool`, which skips the client session.
