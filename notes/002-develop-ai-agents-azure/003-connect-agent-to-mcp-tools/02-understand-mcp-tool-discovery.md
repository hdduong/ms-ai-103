# 02 Understand MCP tool discovery

[Understand MCP tool discovery](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/2-understand-mcp-tool-discovery?pivots=text)

- MCP is a live catalog. The agent asks the server which tools exist. You do not hard-code each API, and you do not redeploy the agent when a tool changes.
- Exam benefits: dynamic discovery, the same tools across models, one auth pattern instead of a key per API.
- The server hosts tools with `@mcp.tool`. The client calls `session.list_tools()`, wraps each one, and the agent calls them.

## Server

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("snowfall")

@mcp.tool()
def recent_snowfall(location: str) -> str:
    """Get recent snowfall totals for a city."""
    return f"{location}: 2 inches"
```
