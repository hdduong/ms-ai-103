# 02 Understand MCP tool discovery

[Understand MCP tool discovery](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/2-understand-mcp-tool-discovery?pivots=text)

- MCP is a live catalog. The agent asks the server which tools exist. You do not hard-code each API, and you do not redeploy the agent when a tool changes.
- Exam benefits: **dynamic tool discovery**, **interoperability across LLMs** (same tools for any model), **standardized security** (one auth pattern instead of a key per API).
- The server hosts tools with `@mcp.tool`. The client calls `session.list_tools()`, wraps each one, and the agent calls them.

## Lab server: `server.py`

Lab: [Extend agents with MCP tools](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html). The inventory server with the lab's data. The client in [03](03-mcp-client-server-setup.md#lab-client-clientpy) starts it, so do not run it in another terminal. Keep `print` off stdout, because stdout carries the MCP messages.

```python
from fastmcp import FastMCP

mcp = FastMCP(name="Inventory")

@mcp.tool()
def get_inventory_levels() -> dict:
    """Return the current inventory count for each product."""
    return {
        "Moisturizer": 6, "Shampoo": 8, "Body Spray": 28, "Hair Gel": 5,
        "Lip Balm": 12, "Skin Serum": 9, "Cleanser": 30, "Conditioner": 3,
        "Setting Powder": 17, "Dry Shampoo": 45,
    }

@mcp.tool()
def get_weekly_sales() -> dict:
    """Return units sold this week for each product."""
    return {
        "Moisturizer": 22, "Shampoo": 18, "Body Spray": 3, "Hair Gel": 2,
        "Lip Balm": 14, "Skin Serum": 19, "Cleanser": 4, "Conditioner": 1,
        "Setting Powder": 13, "Dry Shampoo": 17,
    }

if __name__ == "__main__":
    mcp.run(transport="stdio", show_banner=False)
```
