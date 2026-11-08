# 05 Exercise — extend agents with MCP tools

[Current Microsoft Learn exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html) · [Reviewed instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/03-mcp-integration.md) · [Starter files](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/03-mcp-integration/Python)

Reviewed **2026-11-07** at commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`.

## Two connections in one exercise

| Part | Who communicates with MCP? | Tools seen by the Foundry agent |
| --- | --- | --- |
| Remote Microsoft Learn server | Foundry Agent Service calls the remote HTTPS endpoint | One `MCPTool` describes the server; discovery and remote calls are handled by the service |
| Local inventory server | Your Python application starts a subprocess and uses an MCP `ClientSession` over stdio | One `FunctionTool` per discovered MCP tool; your application dispatches function calls through that session |

The completed examples below combine the starter files and instruction-page code. They add bounded tool loops, explicit approval decisions, conversation cleanup, and a fresh list of outputs for each response. They use the same conversation for every turn and result instead of mixing that state with a separate `previous_response_id` chain.

## 1. Prepare the environment

Create/select a Foundry project, deploy a tool-capable model, and copy its project endpoint. The current lab uses Python **3.13** and the Foundry Toolkit for VS Code. Python 3.14 is not supported by the lab's tested dependency set.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/03-mcp-integration/Python
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv mcp fastmcp
az login
```

Create `.env`:

```dotenv
PROJECT_ENDPOINT=https://<foundry-resource>.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

Your signed-in identity needs access to the project and model. The Microsoft Learn server is public; private MCP servers need their own authentication/connection configuration. Do not confuse that server authentication with the Entra credentials used to call Foundry.

## 2. Remote Microsoft Learn MCP — `agent.py`

The exercise endpoint is `https://learn.microsoft.com/api/mcp`. `require_approval="always"` asks the client to supply approval when the model requests a tool. It does not itself display a human approval UI. The upstream lab approves its public documentation requests automatically; this adaptation prints each request and asks you to approve or deny it.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import MCPTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def main():
    load_dotenv()
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.create_version(
            agent_name="learn-docs-mcp-agent",
            definition=PromptAgentDefinition(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],
                instructions="Use Microsoft Learn MCP tools for current Azure documentation.",
                tools=[MCPTool(server_label="api-specs",
                               server_url="https://learn.microsoft.com/api/mcp",
                               require_approval="always")],
            ),
        )
        conversation = None
        try:
            conversation = client.conversations.create()
            reference = {"agent_reference": {"name": agent.name, "version": agent.version,
                                              "type": "agent_reference"}}
            response = client.responses.create(
                conversation=conversation.id,
                input=("Give me the Azure CLI commands to create an Azure Container App "
                       "with a managed identity. Cite the Microsoft Learn documentation."),
                extra_body=reference,
            )
            for round_number in range(9):
                if response.status == "failed":
                    raise RuntimeError(str(response.error))
                requests = [item for item in response.output
                            if item.type == "mcp_approval_request"]
                if not requests:
                    print(response.output_text)
                    break
                if round_number == 8:
                    raise RuntimeError("Stopped after eight MCP approval rounds.")
                decisions = []
                for request in requests:
                    print(f"\nServer: {request.server_label}\nTool: {request.name}")
                    print(f"Arguments: {request.arguments}")
                    approved = False
                    if request.server_label == "api-specs":
                        approved = input("Allow this documentation request? [y/N] ").lower() == "y"
                    decisions.append({"type": "mcp_approval_response",
                                      "approval_request_id": request.id,
                                      "approve": approved})
                response = client.responses.create(
                    conversation=conversation.id, input=decisions, extra_body=reference,
                )
            else:
                raise RuntimeError("Stopped after eight MCP approval rounds.")
        finally:
            try:
                if conversation is not None:
                    client.conversations.delete(conversation_id=conversation.id)
            finally:
                project.agents.delete_version(agent_name=agent.name,
                                              agent_version=agent.version)


if __name__ == "__main__":
    main()
```

Run `python agent.py`. Approve a documentation retrieval, inspect the answer and its source links, then rerun and deny a request. Approval is specific to the requested operation. Returning `approve=True` without a user decision is an automatic approval policy, even when `require_approval` is `"always"`.

## 3. Local inventory MCP — `server.py`

These are the exercise's complete sample inventory and weekly-sales values. The server exposes two read-only tools with no arguments.

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

The client below launches the server. Do not start a second copy in another terminal. Keep ordinary `print` logging off the server's stdout because stdout carries protocol messages; send diagnostic logs to stderr.

## 4. Discover, register, and dispatch — `client.py`

The inventory tools have no parameters. For a reusable bridge, this adaptation forwards each discovered tool's actual `inputSchema` instead of replacing it with an empty object. `strict=False` allows arbitrary MCP JSON schemas; a production bridge can explicitly normalize supported schemas before enabling strict function validation. `model_dump(by_alias=True)` reads the protocol field names (`inputSchema`, `isError`), which remain stable across the Python SDK's camel-case/snake-case attribute change.

```python
import asyncio
import json
import os
import sys
from contextlib import AsyncExitStack
from pathlib import Path
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import FunctionTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client


async def run_turn(client, session, tool_names, conversation_id, reference, prompt):
    response = client.responses.create(
        conversation=conversation_id, input=prompt, extra_body=reference,
    )
    for round_number in range(9):
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        calls = [item for item in response.output if item.type == "function_call"]
        if not calls:
            return response.output_text
        if round_number == 8:
            raise RuntimeError("Stopped after eight function-call rounds.")
        outputs = []
        for call in calls:
            if call.name not in tool_names:
                output = json.dumps({"error": "Unknown MCP tool."})
            else:
                try:
                    arguments = json.loads(call.arguments)
                    result = await session.call_tool(call.name, arguments)
                    # This lab's tools return text containing JSON. Preserve all text blocks.
                    text = "\n".join(part.text for part in result.content if part.type == "text")
                    output = json.dumps({"is_error": bool(result.model_dump(by_alias=True).get("isError", False)),
                                         "content": text})
                except (ValueError, TypeError) as exc:
                    output = json.dumps({"error": str(exc)})
            outputs.append({"type": "function_call_output", "call_id": call.call_id,
                            "output": output})
        response = client.responses.create(
            conversation=conversation_id, input=outputs, extra_body=reference,
        )
    raise RuntimeError("Stopped after eight function-call rounds.")


async def main():
    load_dotenv()
    server = Path(__file__).resolve().with_name("server.py")
    params = StdioServerParameters(command=sys.executable, args=[str(server)])
    async with AsyncExitStack() as stack:
        read, write = await stack.enter_async_context(stdio_client(params))
        session = await stack.enter_async_context(ClientSession(read, write))
        await session.initialize()
        discovered = (await session.list_tools()).tools
        tool_names = {tool.name for tool in discovered}
        print("Discovered:", sorted(tool_names))
        tools = [FunctionTool(name=tool.name, description=tool.description or tool.name,
                              parameters=tool.model_dump(by_alias=True)["inputSchema"], strict=False)
                 for tool in discovered]
        with (
            DefaultAzureCredential() as credential,
            AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                            credential=credential) as project,
            project.get_openai_client() as client,
        ):
            agent = project.agents.create_version(
                agent_name="inventory-mcp-agent",
                definition=PromptAgentDefinition(
                    model=os.environ["MODEL_DEPLOYMENT_NAME"], tools=tools,
                    instructions=("Use inventory and weekly sales to recommend actions. "
                                  "Restock products with inventory < 10 AND weekly sales > 15. "
                                  "Clearance products with inventory > 20 AND weekly sales < 5. "
                                  "State both counts for each recommendation."),
                ),
            )
            conversation = None
            try:
                conversation = client.conversations.create()
                reference = {"agent_reference": {"name": agent.name, "version": agent.version,
                                                  "type": "agent_reference"}}
                while True:
                    prompt = input("You (exit to stop): ").strip()
                    if prompt.lower() in {"exit", "quit"}:
                        break
                    if prompt:
                        print(await run_turn(client, session, tool_names,
                                             conversation.id, reference, prompt))
            finally:
                try:
                    if conversation is not None:
                        client.conversations.delete(conversation_id=conversation.id)
                finally:
                    project.agents.delete_version(agent_name=agent.name,
                                                  agent_version=agent.version)


if __name__ == "__main__":
    asyncio.run(main())
```

Run `python client.py`, then ask `Which products should I restock and which should go on clearance?` The sample data and the strict `<`/`>` rules yield:

| Action | Product | Inventory | Weekly sales |
| --- | --- | ---: | ---: |
| Restock | Moisturizer | 6 | 22 |
| Restock | Shampoo | 8 | 18 |
| Restock | Skin Serum | 9 | 19 |
| Clearance | Body Spray | 28 | 3 |
| Clearance | Cleanser | 30 | 4 |

Ask a second question in the same session to verify another tool round works. The MCP client discovered a snapshot of tools at startup; if server tool names or schemas change, refresh discovery and update the registered agent definition. Dynamic discovery does not guarantee that every client automatically reloads changed schemas.

## Cleanup and attribution

`exit` closes the local MCP session/subprocess and deletes the conversation and agent version created by that script. The remote example performs equivalent cleanup when its single request finishes. Delete the dedicated project/resource group when finished if it contains only the lab resources.

Adapted from MicrosoftLearning sample code under the **MIT License**; see [Third-party notices](../../../THIRD_PARTY_NOTICES.md). Syntax, SDK model construction, and the local stdio tool interaction can be verified without an Azure run. Remote agent execution still requires your deployed model, project access, and a reachable MCP endpoint.
