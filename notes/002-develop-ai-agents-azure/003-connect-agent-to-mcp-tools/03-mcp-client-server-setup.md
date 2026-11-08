# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

- This unit is the hand-written client. Unit 04 is Foundry calling the server for you.
- Server: `FastMCP("server-name")` and `@mcp.tool`. The server owns the definitions; a client discovers them. Refresh discovery and registered schemas when the catalog changes.
- Client: open a session, `session.list_tools()`, wrap each tool in an async function, call it with `session.call_tool(tool_name, tool_args)`, create one `FunctionTool` definition per discovered tool, and register the list on the agent. Preserve each tool's `inputSchema`; the exercise's empty schema works only for its no-argument tools.

## Client

```python
from mcp import ClientSession
from mcp.client.stdio import stdio_client

stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
stdio, write = stdio_transport
session = await exit_stack.enter_async_context(ClientSession(stdio, write))
await session.initialize()
listed = await session.list_tools()
tools = listed.tools
print([tool.name for tool in tools])

def make_tool_func(tool_name):
    async def tool_func(**kwargs):
        return await session.call_tool(tool_name, kwargs)
    tool_func.__name__ = tool_name
    return tool_func

functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
```

This fragment assumes `exit_stack` and `server_params` exist inside an async function. [The complete exercise](05-exercise-mcp-tools.md) supplies setup, startup, schemas, dispatch, verification, and cleanup.
