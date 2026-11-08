# 02 Understand MCP tool discovery

[Understand MCP tool discovery](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/2-understand-mcp-tool-discovery?pivots=text)

- MCP is a live catalog. The agent asks the server which tools exist. You do not hard-code each API, and you do not redeploy the agent when a tool changes.
- Exam benefits: dynamic discovery, the same tools across models, one auth pattern instead of a key per API.
- The server hosts tools with `@mcp.tool`. The client calls `session.list_tools()`, wraps each one, and the agent calls them.

## Server

The exercise hosts the tools with FastMCP. Lab: [Extend agents with MCP tools](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html).

```python
from fastmcp import FastMCP

mcp = FastMCP(name="Inventory")

@mcp.tool()
def get_inventory_levels() -> dict:
    """Return the current stock count for each product."""
    return {"moisturizer": 6, "shampoo": 8, "body_spray": 28}

@mcp.tool()
def get_weekly_sales() -> dict:
    """Return units sold this week for each product."""
    return {"moisturizer": 20, "shampoo": 4, "body_spray": 3}

mcp.run(show_banner=False)
```
