# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: Learn describes `"always"` as the default. Set it explicitly when you want the application to decide before a call executes. `"never"` allows execution without that approval round. An `mcp_approval_request` is a protocol pause; it becomes human approval only if your application actually asks a person.

## Lab: remote Microsoft Learn MCP

Lab: [Extend agents with MCP tools](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html). Same `.env` and packages as the [client lab](03-mcp-client-server-setup.md#lab-client-clientpy). `require_approval="always"` means the first response is usually an `mcp_approval_request`, not the answer.

### Configure the hosted connection

Save this complete example as `remote_agent.py`. Unlike the local bridge, it registers one `MCPTool` with a reachable server URL. Foundry discovers and calls the server; this client only handles the approval decisions.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import MCPTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def answer_approvals(client, response, reference):
    for round_number in range(9):
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        requests = [item for item in response.output if item.type == "mcp_approval_request"]
        if not requests:
            return response.output_text
        if round_number == 8:
            raise RuntimeError("Stopped after eight approval rounds.")
        decisions = []
        for request in requests:
            print(f"Server: {request.server_label}  Tool: {request.name}")
            print("Arguments:", request.arguments)
            approve = (
                request.server_label == "api-specs"
                and input("Approve? [y/N] ").strip().lower() == "y"
            )
            decisions.append({
                "type": "mcp_approval_response",
                "approval_request_id": request.id,
                "approve": approve,
            })
        response = client.responses.create(
            input=decisions,
            previous_response_id=response.id,
            extra_body=reference,
        )
    raise RuntimeError("Stopped after eight approval rounds.")


def main():
    load_dotenv()
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.create_version(
            agent_name="learn-docs-agent",
            definition=PromptAgentDefinition(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
                instructions="Use the Microsoft Learn MCP tools to answer Azure questions.",
                tools=[MCPTool(
                    server_label="api-specs",
                    server_url="https://learn.microsoft.com/api/mcp",
                    require_approval="always",
                )],
            ),
        )
        conversation = None
        try:
            conversation = client.conversations.create()
            reference = {"agent_reference": {
                "name": agent.name, "version": agent.version,
                "type": "agent_reference",
            }}
            response = client.responses.create(
                conversation=conversation.id,
                input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
                extra_body=reference,
            )
            print(answer_approvals(client, response, reference))
        finally:
            try:
                if conversation is not None:
                    client.conversations.delete(conversation_id=conversation.id)
            finally:
                project.agents.delete_version(
                    agent_name=agent.name, agent_version=agent.version,
                )


if __name__ == "__main__":
    main()
```

### Understand the pause and continuation

`require_approval="always"` makes the service return the requested server, tool, arguments, and approval ID. The client displays those details and returns a decision with the same `approval_request_id`. `previous_response_id` resumes that paused response, following the remote-MCP exercise pattern. This is a different continuation shape from the local function loop's explicit conversation input.

The official exercise sends `approve=True` automatically. The prompt above is an adaptation that makes the decision visible to the learner and defaults to denial. Repeat the program, deny a request, and check that the answer acknowledges the missing documentation lookup instead of claiming the tool ran. Approving this documentation lookup retrieves command text; the program does not execute the suggested Azure CLI commands.

```bash
python remote_agent.py
```
