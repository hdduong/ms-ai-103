# 08 Test, deploy, and integrate agents

[Test, deploy, and integrate agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/8-test-deploy-integrate?pivots=text)

- Playground checks: happy path, edge cases, out-of-scope requests, more than one turn, and that the right tool runs.
- **Save** in the portal, or **Save to Foundry** in VS Code, writes the prompt agent into the project.
- A hosted agent deploys from **+ Build > Deploy to Microsoft Foundry**. **Code (ZIP Code Deploy)** ships source with no image. **Container (BYOI)** uses an image already in Azure Container Registry.
- Creating the agent also creates a stable endpoint, its own Entra identity, and an agent blueprint. Project RBAC does not transfer. A tool that worked in the playground fails after publish until that identity has the role.
- Callers need **Foundry User** on the agent. API keys are not supported. A missing role is **403**.
- The published endpoint is stateless Responses. The client keeps the history. The URL stays the same when you save a new version.
- In VS Code, **View Code** on the agent writes a sample that signs in and calls this endpoint.

## Published endpoint

The agent's **Details** tab shows the Entra agent identity, the blueprint, and the Responses endpoint.

![it-support-agent Details: Running status, active version, Entra agent identity and blueprint, and the Responses protocol endpoint. IDs are masked.](images/08-agent-details.png)

```text
POST https://{resource-name}.services.ai.azure.com/api/projects/<project-name>/agents/healthcare-assistant/endpoint/protocols/openai/responses?api-version=2025-11-15-preview
Authorization: Bearer <token from az account get-access-token --resource https://ai.azure.com>
Content-Type: application/json

{"input":"Say hello"}
```

`model` is not on this call. The agent already has the deployment.

The same call from Python. The token scope is `https://ai.azure.com/.default`, and the caller needs **Foundry User** on the agent.

```python
import requests
from azure.identity import DefaultAzureCredential

token = DefaultAzureCredential().get_token("https://ai.azure.com/.default").token
url = (
    "https://{resource-name}.services.ai.azure.com/api/projects/<project-name>"
    "/agents/it-support-agent/endpoint/protocols/openai/responses"
    "?api-version=2025-11-15-preview"
)
response = requests.post(
    url,
    headers={"Authorization": f"Bearer {token}"},
    json={"input": "Say hello"},
    timeout=60,
)
response.raise_for_status()
print(response.status_code, response.json())
```

## Lab client: chat with the portal agent

From the [portal and VS Code lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/01-build-agent-portal-and-vscode.html). This goes through the project SDK, not the published endpoint, so the conversation keeps the history.

```bash
pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
AGENT_NAME=it-support-agent
```

Code Interpreter charts come back as `container_file_citation` annotations. Download them with `containers.files.content`.

```python
import os
from pathlib import Path
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()
project_client = AIProjectClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()
agent = project_client.agents.get(agent_name=os.environ["AGENT_NAME"])
conversation = openai_client.conversations.create(items=[])

while True:
    prompt = input("You (exit to stop): ").strip()
    if prompt.lower() in {"exit", "quit"}:
        break
    openai_client.conversations.items.create(
        conversation_id=conversation.id,
        items=[{"type": "message", "role": "user", "content": prompt}],
    )
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
    print("Agent:", response.output_text)
    for item in response.output:
        if item.type != "message":
            continue
        for content in item.content:
            for annotation in getattr(content, "annotations", None) or []:
                if annotation.type == "container_file_citation":
                    data = openai_client.containers.files.content.retrieve(
                        container_id=annotation.container_id,
                        file_id=annotation.file_id,
                    )
                    Path(Path(annotation.filename).name).write_bytes(data.read())
                    print("Saved", annotation.filename)

openai_client.conversations.delete(conversation_id=conversation.id)
```

Use the test prompts from [04](04-build-agent-azure-portal.md#lab-it-support-agent). Check policy answers against `IT_Policy.txt` and numbers against the CSV.
