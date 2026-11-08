# 09 Exercise — Build an agent with the portal and VS Code

[Current exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/01-build-agent-portal-and-vscode.html) · [Pinned instructions and complete upstream client](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/01-build-agent-portal-and-vscode.md)

Reviewed **2026-11-07**. Build `it-support-agent` in the **new Foundry** portal, attach policy data and a CSV, then call the same saved agent through the project SDK. The exercise's Python starter file is empty at the reviewed revision; the implementation is supplied in the exercise instructions. The completed client below is a compact adaptation of that implementation.

## Prerequisites and agent configuration

Use an Azure subscription with quota and permissions, VS Code with **Foundry Toolkit**, Git, Azure CLI, and **Python 3.13**. The current exercise was tested with Python 3.13.12 and says Python 3.14 isn't supported by all lab dependencies yet.

Create a project and an agent named `it-support-agent`. Select a deployed model, add **File search** and **Code interpreter**, and save these instructions:

```text
You are an IT Support Agent for Contoso Corporation.
Help employees with technical issues and IT policy questions.
Use the uploaded policy documentation for policy answers.
Be professional and helpful. If you don't know, admit it and suggest IT support.
Collect the required details before proposing a support ticket.
```

| File | Attach to | What it demonstrates |
| --- | --- | --- |
| [IT_Policy.txt](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt) | File search / uploaded grounding files | Retrieve fictional Contoso policies |
| [system_performance.csv](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv) | Code interpreter **+ Files** | Analyze simulated metrics and create charts |

Wait for file indexing, test both tools in the playground, then select the project under **Microsoft Foundry Resources > Set Default Project** in VS Code. Open the saved prompt agent in Agent Builder.

## Prepare the local client

Clone once, or reuse your existing checkout. The pinned commit makes the source/version used in this note reproducible.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/01-build-agent-portal-and-vscode/Python
python -m venv .venv
```

Activate on macOS/Linux with `source .venv/bin/activate`; on PowerShell use `.\.venv\Scripts\Activate.ps1`. Install the v2 SDK used by the adjacent current labs; `openai<3` matches upstream's compatibility constraint:

```bash
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

Create a local `.env` and keep it out of Git:

```dotenv
PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<project>
AGENT_NAME=it-support-agent
```

Copy the actual **project endpoint** from Foundry or the VS Code project menu. A model deployment endpoint, old project connection string, and published-agent endpoint aren't interchangeable with this URL. The caller needs permission to use this project and agent; no API key is stored in this client.

## Completed client: `agent_with_functions.py`

The code uses the agent's existing model and tools. A server-side conversation preserves context between turns. File Search and Code Interpreter execute in the service; their generated files must be downloaded if you want local copies.

```python
import base64
import os
from pathlib import Path

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

OUTPUT_DIR = Path("agent_outputs")


def save_bytes(data: bytes, filename: str) -> Path:
    OUTPUT_DIR.mkdir(exist_ok=True)
    name = Path(filename).name
    if name in {"", ".", ".."}:
        name = "generated.bin"
    path = OUTPUT_DIR / name
    stem, suffix = path.stem, path.suffix
    counter = 1
    while path.exists():
        path = OUTPUT_DIR / f"{stem}_{counter}{suffix}"
        counter += 1
    path.write_bytes(data)
    return path


def display_response(client, response):
    print(f"Agent: {response.output_text}")
    downloaded = set()
    for item in response.output or []:
        if item.type == "message":
            for content in item.content:
                if content.type != "output_text":
                    continue
                for annotation in content.annotations or []:
                    if annotation.type != "container_file_citation":
                        continue
                    key = (annotation.container_id, annotation.file_id)
                    if key in downloaded:
                        continue
                    data = client.containers.files.content.retrieve(
                        container_id=annotation.container_id,
                        file_id=annotation.file_id,
                    )
                    path = save_bytes(
                        data.read(), annotation.filename or "generated.bin"
                    )
                    downloaded.add(key)
                    print(f"Saved generated file: {path}")
        elif item.type == "image":
            # Retain the inline-image case demonstrated by the upstream client.
            data = getattr(getattr(item, "image", None), "data", None)
            if data:
                path = save_bytes(base64.b64decode(data), "chart.png")
                print(f"Saved image: {path}")


def main():
    load_dotenv()
    endpoint = os.environ["PROJECT_ENDPOINT"]
    agent_name = os.getenv("AGENT_NAME", "it-support-agent")
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=endpoint, credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.get(agent_name=agent_name)
        conversation = client.conversations.create(items=[])
        reference = {"agent_reference": {
            "name": agent.name, "type": "agent_reference"
        }}
        try:
            while True:
                prompt = input("You (exit to stop): ").strip()
                if prompt.lower() in {"exit", "quit", "bye"}:
                    break
                if not prompt:
                    continue
                client.conversations.items.create(
                    conversation_id=conversation.id,
                    items=[{"type": "message", "role": "user", "content": prompt}],
                )
                response = client.responses.create(
                    conversation=conversation.id, input="", extra_body=reference
                )
                if response.status == "failed":
                    raise RuntimeError(f"Agent response failed: {response.error}")
                display_response(client, response)
        finally:
            # This client owns the conversation, but the portal agent is reusable.
            client.conversations.delete(conversation_id=conversation.id)


if __name__ == "__main__":
    main()
```

Run:

```bash
python agent_with_functions.py
```

## Verify behavior

```text
What's the policy for password resets?
Analyze the system performance data and identify when CPU usage exceeded 80%.
Create a line chart showing memory usage over time.
What are the average, minimum, and maximum disk usage values?
```

Check policy claims against `IT_Policy.txt`, numerical answers against the CSV, and chart/file output under `agent_outputs`. A response containing a remote sandbox link doesn't save a local file; the `container_file_citation` download handles that step. The upstream implementation also rewrites those citations in displayed text; this shorter version prints each saved local path separately.

## Cleanup and takeaways

The client deletes the conversation it creates and closes its clients. It preserves the portal-created agent and uploaded data so you can continue the lessons. When finished, remove the lab agent/files and dedicated project resources, or review and delete the dedicated resource group. Model usage and Code Interpreter may incur charges.

Remember: `AIProjectClient` manages the project/agent; `get_openai_client()` supplies Responses, Conversations, and container-file APIs. `agent_reference` chooses a saved agent. Don't replace this project SDK call with a published endpoint call and assume the same conversation behavior.

Code adapted from MicrosoftLearning/mslearn-ai-agents at `4d6a32629722927bd5f1034e0b5396e0da4e962c` (2026-11-04), with context-manager cleanup, failure checks, and a compact file downloader. See [third-party notices](../../../THIRD_PARTY_NOTICES.md).
