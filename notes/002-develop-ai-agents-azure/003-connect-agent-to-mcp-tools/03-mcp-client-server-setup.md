# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

- This unit is the hand-written client. Unit 04 is Foundry calling the server for you.
- Server: `FastMCP("server-name")` and `@mcp.tool`. Definitions stay on the server, so a new tool does not require a new agent deploy.
- Client: open a session, `session.list_tools()`, wrap each tool in an async function, call it with `session.call_tool(tool_name, tool_args)`, bundle the wrappers in a `FunctionTool`, register that on the agent.

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
