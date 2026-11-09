# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: `"always"` is the default and pauses for a person. `"never"` runs the tool. If approval is required, the response has `mcp_approval_request`. Reply with `mcp_approval_response` (`approval_request_id` and `approve`).

## Lab: remote Microsoft Learn MCP

Lab: [Extend agents with MCP tools](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html). Same `.env` and packages as the [client lab](03-mcp-client-server-setup.md#lab-client-clientpy). `require_approval="always"` means the first response is usually an `mcp_approval_request`, not the answer.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import MCPTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from openai.types.responses.response_input_param import McpApprovalResponse

load_dotenv()
project_client = AIProjectClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()

mcp_tool = MCPTool(
    server_label="api-specs",
    server_url="https://learn.microsoft.com/api/mcp",
    require_approval="always",
)
agent = project_client.agents.create_version(
    agent_name="MyAgent",
    definition=PromptAgentDefinition(
        model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
        instructions="You are a helpful agent that can use MCP tools to assist users. Use the available MCP tools to answer questions and perform tasks.",
        tools=[mcp_tool],
    ),
)
conversation = openai_client.conversations.create()
response = openai_client.responses.create(
    conversation=conversation.id,
    input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
)
while True:
    input_list = []
    for item in response.output:
        if item.type == "mcp_approval_request" and item.server_label == "api-specs" and item.id:
            print(f"Tool: {item.name}  Arguments: {item.arguments}")
            input_list.append(
                McpApprovalResponse(
                    type="mcp_approval_response",
                    approve=input("Approve? [y/N] ").lower() == "y",
                    approval_request_id=item.id,
                )
            )
    if not input_list:
        break
    response = openai_client.responses.create(
        input=input_list,
        previous_response_id=response.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    )
print(response.output_text)

openai_client.conversations.delete(conversation_id=conversation.id)
project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
```

- The lab sends `approve=True` without asking. Asking with `input()` is the human-in-the-loop version.
- Deny once and see how the answer changes when the tool does not run.
