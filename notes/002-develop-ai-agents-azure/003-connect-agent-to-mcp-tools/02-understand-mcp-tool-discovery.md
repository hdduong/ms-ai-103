# 02 Understand MCP tool discovery

[Understand MCP tool discovery](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/2-understand-mcp-tool-discovery?pivots=text)

- MCP lets a client discover a server's tool names, descriptions, and parameter schemas. A client can avoid hand-writing each API adapter, but it must refresh discovery when tools change. The local exercise registers discovered schemas in an agent version, so a schema change can require an updated definition.
- Benefits: dynamic discovery and reusable tool interfaces across clients/models. Authentication still depends on each server and its deployment; MCP does not eliminate authorization or all service-specific credentials.
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

The small server above is illustrative. [The complete exercise](05-exercise-mcp-tools.md) includes the upstream inventory, runnable local client, and remote Microsoft Learn MCP example.
