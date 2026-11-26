# 04 Use Azure AI agents with MCP servers

[Use Azure AI agents with MCP servers](https://learn.microsoft.com/en-us/training/modules/connect-agent-to-mcp-tools/4-use-azure-ai-agents-with-mcp?pivots=text)

- Foundry calls a remote MCP server for you. No `list_tools` and no hand-written wrappers.
- `MCPTool`: `server_label`, `server_url`, optional `allowed_tools`, optional `require_approval`. Headers carry API keys or OAuth tokens. One tool object per server.
- Approval: Learn describes `"always"` as the default. Set it explicitly when you want the application to decide before a call executes. `"never"` allows execution without that approval round. An `mcp_approval_request` is a protocol pause; it becomes human approval only if your application actually asks a person.

## Lab: remote Microsoft Learn MCP

Lab: [Extend agents with MCP tools](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html). Same `.env` and packages as the [client lab](03-mcp-client-server-setup.md#lab-client-clientpy). `require_approval="always"` means the first response is usually an `mcp_approval_request`, not the answer.

### Configure the hosted connection

Save as `remote_agent.py`. Foundry discovers and calls the MCP server itself; this client only answers the approval requests.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import MCPTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def answer_approvals(client, response, reference):
    # Keep answering approval requests until the final answer comes back
    for round_number in range(9):
        # The run failed on the service side
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        # MCP calls the agent wants to make, waiting for a yes/no
        requests = [item for item in response.output if item.type == "mcp_approval_request"]
        # Nothing waiting: this response is the answer
        if not requests:
            return response.output_text
        # Safety stop so a chatty agent can't loop forever
        if round_number == 8:
            raise RuntimeError("Stopped after eight approval rounds.")
        decisions = []
        for request in requests:
            # Show which tool it wants to call, and with what arguments
            print(f"Server: {request.server_label}  Tool: {request.name}")
            print("Arguments:", request.arguments)
            # Yes only for our server and only if you type y; anything else is no
            approve = (
                request.server_label == "api-specs"
                and input("Approve? [y/N] ").strip().lower() == "y"
            )
            # Each decision points back at its request by id
            decisions.append({
                "type": "mcp_approval_response",
                "approval_request_id": request.id,
                "approve": approve,
            })
        # Send the decisions; previous_response_id resumes the paused response
        response = client.responses.create(
            input=decisions,
            previous_response_id=response.id,
            extra_body=reference,
        )
    raise RuntimeError("Stopped after eight approval rounds.")


def main():
    # Read PROJECT_ENDPOINT and MODEL_DEPLOYMENT_NAME from .env
    load_dotenv()
    # Sign in with az login, open the project, get the OpenAI-style client
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        # Save an agent version whose only tool is the Learn MCP server
        agent = project.agents.create_version(
            agent_name="learn-docs-agent",
            definition=PromptAgentDefinition(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
                instructions="Use the Microsoft Learn MCP tools to answer Azure questions.",
                tools=[MCPTool(
                    server_label="api-specs",  # name that tags this server's calls
                    server_url="https://learn.microsoft.com/api/mcp",  # Foundry calls it, not you
                    require_approval="always",  # pause before every call
                )],
            ),
        )
        conversation = None
        try:
            # A fresh conversation for this run
            conversation = client.conversations.create()
            # Run this saved agent version, not a bare model
            reference = {"agent_reference": {
                "name": agent.name, "version": agent.version,
                "type": "agent_reference",
            }}
            # Ask; the first reply is usually an approval request, not the answer
            response = client.responses.create(
                conversation=conversation.id,
                input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
                extra_body=reference,
            )
            print(answer_approvals(client, response, reference))
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
    main()
```

```bash
python remote_agent.py
```

- The official lab sends `approve=True` without asking. Here you decide.
- Run it twice: approve once, deny once. The denied run should say it couldn't check the docs, not pretend it did. Either way the program only prints CLI commands; it never runs them.
