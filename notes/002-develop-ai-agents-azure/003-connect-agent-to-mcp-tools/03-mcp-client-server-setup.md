# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

- This unit is the hand-written client. Unit 04 is Foundry calling the server for you.
- Server: `FastMCP("server-name")` and `@mcp.tool`. Definitions stay on the server, so a new tool does not require a new agent deploy.
- Client: open a session, `session.list_tools()`, wrap each tool in an async function, call it with `session.call_tool(tool_name, tool_args)`, bundle the wrappers in a `FunctionTool`, register that on the agent.

## Client

```python
listed = await session.list_tools()
result = await session.call_tool("recent_snowfall", {"location": "Denver"})
print(result)
```
