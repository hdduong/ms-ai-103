# 03 Integrate agent tools using an MCP server and client

[Integrate agent tools using an MCP server and client](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/3-mcp-client-server-setup?pivots=text)

- This unit is the hand-written client. Unit 04 is Foundry calling the server for you.
- Server: `FastMCP("server-name")` and `@mcp.tool`. Definitions stay on the server, so a new tool does not require a new agent deploy.
- Client: open a session, `session.list_tools()`, wrap each tool in an async function, call it with `session.call_tool(tool_name, tool_args)`, bundle the wrappers in a `FunctionTool`, register that on the agent.

## Lab client: `client.py`

From the [MCP lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html). It starts [`server.py`](02-understand-mcp-tool-discovery.md#lab-server-serverpy) over stdio, turns each discovered tool into a `FunctionTool`, and runs the calls the model asks for through the MCP session.

```bash
pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv mcp fastmcp
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

```python
import asyncio
import json
import os
import sys
from contextlib import AsyncExitStack
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import FunctionTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    load_dotenv()
    async with AsyncExitStack() as exit_stack:
        server_params = StdioServerParameters(command=sys.executable, args=["server.py"])
        stdio, write = await exit_stack.enter_async_context(stdio_client(server_params))
        session = await exit_stack.enter_async_context(ClientSession(stdio, write))
        await session.initialize()
        tools = (await session.list_tools()).tools
        print("Discovered:", [tool.name for tool in tools])

        def make_tool_func(tool_name):
            async def tool_func(**kwargs):
                return await session.call_tool(tool_name, kwargs)
            return tool_func

        functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
        mcp_function_tools = [
            FunctionTool(
                name=tool.name,
                description=tool.description or tool.name,
                parameters=tool.model_dump(by_alias=True)["inputSchema"],
                strict=False,
            )
            for tool in tools
        ]

        project_client = AIProjectClient(
            endpoint=os.environ["PROJECT_ENDPOINT"],
            credential=DefaultAzureCredential(),
        )
        openai_client = project_client.get_openai_client()
        agent = project_client.agents.create_version(
            agent_name="inventory-agent",
            definition=PromptAgentDefinition(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
                instructions="""
                You are an inventory assistant. Here are some general guidelines:
                - Recommend restock if item inventory < 10  and weekly sales > 15
                - Recommend clearance if item inventory > 20 and weekly sales < 5
                """,
                tools=mcp_function_tools,
            ),
        )
        reference = {"agent_reference": {"name": agent.name, "type": "agent_reference"}}
        conversation = openai_client.conversations.create()

        while True:
            prompt = input("You (exit to stop): ").strip()
            if prompt.lower() in {"exit", "quit"}:
                break
            response = openai_client.responses.create(
                conversation=conversation.id,
                input=prompt,
                extra_body=reference,
            )
            for _ in range(8):
                calls = [item for item in response.output if item.type == "function_call"]
                if not calls:
                    break
                outputs = []
                for call in calls:
                    result = await functions_dict[call.name](**json.loads(call.arguments))
                    text = "\n".join(part.text for part in result.content if part.type == "text")
                    outputs.append({"type": "function_call_output", "call_id": call.call_id, "output": text})
                response = openai_client.responses.create(
                    conversation=conversation.id,
                    input=outputs,
                    extra_body=reference,
                )
            print("Agent:", response.output_text)

        openai_client.conversations.delete(conversation_id=conversation.id)
        project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)

asyncio.run(main())
```

- `tool.model_dump(by_alias=True)["inputSchema"]` passes each tool's real schema. The lab writes an empty schema, which only works because both tools take no arguments.
- Ask `Are there any products that should be restocked?` and then about clearance. With the lab data: restock Moisturizer (6, 22), Shampoo (8, 18), Skin Serum (9, 19). Clearance Body Spray (28, 3), Cleanser (30, 4).
- The client lists tools once at start. If the server changes its tools, restart the client.
