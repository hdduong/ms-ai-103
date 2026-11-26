# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

A local MCP server cannot be called directly by the hosted Foundry agent. Your application provides the bridge: discover the server's tools, describe them to the model as function tools, then forward requested calls to the MCP session. [Unit 04](04-use-azure-ai-agents-with-mcp.md) shows the alternative in which Foundry calls a remote MCP server directly.

## Lab client: `client.py`

The [MCP exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html) uses the inventory server from [unit 02](02-understand-mcp-tool-discovery.md#lab-server-serverpy). Save that `server.py` and the three Python blocks below as one `client.py` in the same directory. The client starts the server; do not start a second copy manually.

Create/select a Foundry project and deploy `gpt-5` as in the exercise, using your actual deployment name below. Use Python 3.13, the lab's specified version, and a local virtual environment:

```bash
# Create a virtual environment for the lab
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
# Install the lab packages, the MCP SDK, and FastMCP
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv mcp fastmcp
# Sign in so DefaultAzureCredential can use your account
az login
```

Create `.env` next to the scripts:

```dotenv
# Project endpoint from the Foundry project home page
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
# Name of your gpt-5 deployment, used as the agent's model
MODEL_DEPLOYMENT_NAME=gpt-5
```

### Discover the contract and build the bridge

Start `client.py` with this block:

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


async def discover_tools(session):
    # Ask the server for its tools: names, descriptions, input schemas
    discovered = (await session.list_tools()).tools
    print("Discovered:", [tool.name for tool in discovered])

    # Factory so each forwarder keeps its own tool name (no late binding)
    def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            # Run the named tool on the MCP server with the model's arguments
            return await session.call_tool(tool_name, kwargs)
        return tool_func

    # Lookup from tool name to forwarder, used when the model calls one
    functions = {tool.name: make_tool_func(tool.name) for tool in discovered}
    # Describe each MCP tool to the model as a Foundry function tool
    definitions = [
        FunctionTool(
            name=tool.name,
            description=tool.description or tool.name,
            parameters=tool.model_dump(by_alias=True)["inputSchema"],  # the server's real input schema
            strict=False,  # MCP schemas may not meet strict-mode rules
        )
        for tool in discovered
    ]
    return functions, definitions
```

The actual `inputSchema` is preserved, so this bridge also handles tools with arguments. The exercise's empty schema happens to fit its two parameterless tools, but would lose the contract of an argument-taking tool.

`model_dump(by_alias=True)` uses MCP's protocol field names. This works with SDK versions that expose Python attributes such as `input_schema` as well as versions using `inputSchema`.

### Send every tool result back to the model

Append this block to `client.py`:

```python
async def finish_turn(client, conversation_id, reference, functions, prompt):
    # Send the user's prompt to the saved agent in this conversation
    response = client.responses.create(
        conversation=conversation_id, input=prompt, extra_body=reference,
    )
    # Keep running requested tools until the agent gives a final answer
    for round_number in range(9):
        # The run failed on the service side
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        # Function calls the model wants this client to run
        calls = [item for item in response.output if item.type == "function_call"]
        # No calls: this response is the answer
        if not calls:
            return response.output_text
        # Safety stop so a chatty agent can't loop forever
        if round_number == 8:
            raise RuntimeError("Stopped after eight MCP tool rounds.")
        outputs = []
        for call in calls:
            # Find the forwarder for the requested tool
            function = functions.get(call.name)
            # Unknown tool name: answer with an error result, don't crash
            if function is None:
                payload = {"isError": True, "error": f"Unknown tool: {call.name}"}
            else:
                try:
                    # Parse the JSON arguments and run the tool over MCP
                    result = await function(**json.loads(call.arguments))
                    # Keep the whole MCP result: content, structured data, isError
                    payload = result.model_dump(by_alias=True, exclude_none=True)
                except (ValueError, TypeError) as exc:
                    # Bad arguments become an error result instead of a crash
                    payload = {"isError": True, "error": str(exc)}
            # Each output points back at its call by call_id
            outputs.append({
                "type": "function_call_output",
                "call_id": call.call_id,
                "output": json.dumps(payload),
            })
        # Send the results back; the model answers or asks for more tools
        response = client.responses.create(
            conversation=conversation_id, input=outputs, extra_body=reference,
        )
    raise RuntimeError("Stopped after eight MCP tool rounds.")
```

An error must not be presented as successful inventory data. The loop accepts a final answer after the eighth tool round and reports an error if the model requests a ninth; it never silently prints a partial turn as complete.

### Keep the server and conversation alive for the chat

Append the final block to `client.py`:

```python
async def main():
    # Read PROJECT_ENDPOINT and MODEL_DEPLOYMENT_NAME from .env
    load_dotenv()
    # Exit stack closes the session and stops the server when main ends
    async with AsyncExitStack() as exit_stack:
        # How to launch server.py (beside this file) with this same Python
        server_params = StdioServerParameters(
            command=sys.executable,
            args=[str(Path(__file__).with_name("server.py"))],
        )
        # Launch the server and get streams over its stdin/stdout
        read, write = await exit_stack.enter_async_context(stdio_client(server_params))
        # Wrap the streams in an MCP client session
        session = await exit_stack.enter_async_context(ClientSession(read, write))
        # MCP handshake; required before list_tools or call_tool
        await session.initialize()
        # Discover tools once at startup; tool changes need a restart
        functions, definitions = await discover_tools(session)

        # Sign in with az login, open the project, get the OpenAI-style client
        with (
            DefaultAzureCredential() as credential,
            AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                            credential=credential) as project,
            project.get_openai_client() as client,
        ):
            # Save an agent version; the restock rule lives in its instructions
            agent = project.agents.create_version(
                agent_name="inventory-agent",
                definition=PromptAgentDefinition(
                    model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
                    instructions=(
                        "You are an inventory assistant. Use inventory and weekly sales. "
                        "Recommend restock when inventory < 10 and weekly sales > 15. "
                        "Recommend clearance when inventory > 20 and weekly sales < 5."
                    ),
                    tools=definitions,  # discovered MCP tools, as function tools
                ),
            )
            conversation = None
            try:
                # One conversation keeps context across the whole chat
                conversation = client.conversations.create()
                # Run this saved agent version, not a bare model
                reference = {"agent_reference": {
                    "name": agent.name, "version": agent.version,
                    "type": "agent_reference",
                }}
                # Chat loop: one turn per prompt until you type exit
                while True:
                    prompt = input("You (exit to stop): ").strip()
                    if prompt.lower() in {"exit", "quit"}:
                        break
                    if prompt:
                        # Run the turn, including any MCP tool rounds
                        answer = await finish_turn(
                            client, conversation.id, reference, functions, prompt,
                        )
                        print("Agent:", answer)
            finally:
                # Clean up even on errors: the conversation, then the agent version
                try:
                    if conversation is not None:
                        client.conversations.delete(conversation_id=conversation.id)
                finally:
                    project.agents.delete_version(
                        agent_name=agent.name, agent_version=agent.version,
                    )


if __name__ == "__main__":
    asyncio.run(main())
```

The MCP session is asynchronous because it exchanges messages with a subprocess. The Foundry calls and console input here remain synchronous, which is sufficient for this single-user teaching example.

### Check the business rule against the source data

```bash
python client.py
```

Ask `Are there any products that should be restocked?`, then `Which products should go on clearance?` Compare the response with the server dictionaries:

| Action | Products and their `(inventory, weekly sales)` |
| --- | --- |
| Restock | Moisturizer `(6, 22)`, Shampoo `(8, 18)`, Skin Serum `(9, 19)` |
| Clearance | Body Spray `(28, 3)`, Cleanser `(30, 4)` |

The model recommends an action; this server exposes only read operations and does not place an order or change a price. The completed loop, real-schema forwarding, and error preservation are adaptations to the exercise starter.
