# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: `always` is the default and pauses for a person. `never` runs the tool. If approval is required, the response has `mcp_approval_request`. Reply with `mcp_approval_response` (`approval_request_id` and `approve`).

## Remote MCP tool

Lab server is the Microsoft Learn docs MCP endpoint. `require_approval="always"` means the first response is an `mcp_approval_request`, not the answer.

```python
from azure.ai.projects.models import MCPTool, PromptAgentDefinition
from openai.types.responses.response_input_param import McpApprovalResponse

mcp_tool = MCPTool(
    server_label="api-specs",
    server_url="https://learn.microsoft.com/api/mcp",
    require_approval="always",
)
agent = project_client.agents.create_version(
    agent_name="MyAgent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="Use the available MCP tools to answer questions.",
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
            input_list.append(
                McpApprovalResponse(
                    type="mcp_approval_response",
                    approve=True,
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
```
