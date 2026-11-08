# 06 Exercise — integrate an agent with Foundry IQ

[Current Microsoft Learn exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04-integrate-agent-with-foundry-iq.html) · [Reviewed instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/04-integrate-agent-with-foundry-iq.md) · [Starter client](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/04-integrate-agent-with-foundry-iq/Python)

Reviewed **2026-11-07** against commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`.

## What you build

A `product-expert-agent` answers Contoso camping-product questions using three PDFs in Blob Storage. Foundry IQ connects that source to an Azure AI Search knowledge base. A Python application calls the saved agent, shows requested knowledge-base operations, and sends your approval or denial back through the conversation.

The exercise creates resources and the knowledge connection in the portal. Its Python code connects to that saved configuration; it does not create the Search service, upload the documents, or recreate the knowledge base.

## 1. Create the resources and knowledge base

Use the **new Foundry** experience. Region/model availability and capacity vary, so use a supported region with capacity for your model, Search, and Storage.

1. Create a Foundry project and an agent named `product-expert-agent`. Select an available deployed model; the lab uses the default model, such as `gpt-5`.
2. In the agent's **Knowledge** section choose **Add → Connect to Foundry IQ**. Create/select an Azure AI Search resource in the project's resource group and region. The lab suggests Free if available, otherwise Basic; check the tier's current feature availability before creating it.
3. Create a Standard/LRS Storage account in that resource group/region and a container named `contosoproducts`.
4. Download and extract the [pinned sample product ZIP](https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip); upload all three PDFs to the container.
5. In the Search resource, follow the exercise's **Security + networking → Keys → API Access control → Both** setting. This permits its key-based lab connection alongside role-based access. Portal labels can vary; the required behavior is that the lab's key connection is allowed.
6. Return to Foundry **Knowledge** and create a knowledge base. Add the Blob source with the values below, then save the base and wait for its source to become active.
7. Where the portal asks to configure/manage the Search connection, use the lab's **Key** authentication and save the Search key in that connection. Attach the knowledge base to `product-expert-agent` and save.

| Setting | Exercise value |
| --- | --- |
| Source name | `ks-contosoproducts` |
| Description | `Contoso product catalog items` |
| Container | `contosoproducts` |
| Source authentication | API Key, as used in the current exercise |
| Content extraction mode | `minimal` |
| Embedding model | Available deployment, such as `text-embedding-3-small` |
| Chat completions model | Available deployment, such as `gpt-5` |

The portal manages ingestion for this Blob knowledge source. The current [Search documentation](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-blob) explains that it creates an index, data source, skillset, and indexer. “No index to maintain manually” does not mean that retrieval bypasses indexing or immediately sees every file change.

Set these exercise instructions on the agent:

```text
You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products.
You must ALWAYS search the knowledge base to answer questions about our products or product catalog.
Provide detailed, accurate information and always cite your sources.
If you don't find relevant information in the knowledge base, say so clearly.
```

## 2. Enable approval on the correct tool

This step is needed to observe the approval loop in the client. A portal-created knowledge connection may initially auto-approve retrieval.

1. Open Foundry Toolkit in VS Code, sign in, and select the project (**Set Foundry Project** or the corresponding project-selection command in your installed extension).
2. Open `product-expert-agent` under **Agents → Prompt Agents**.
3. In **Tools**, find the tool whose name begins **`kb-knowledgebase...`**. This is the actual Foundry IQ MCP tool.
4. Open that tool's **… → Configure**, choose **Never auto-approve tools**, then **Deploy/Save** the agent configuration.

Changing the separate **Web search** or **Azure AI Search** tool will not set approval for this knowledge-base tool. The example ID shown in the screenshots is illustrative; use the tool actually attached to your agent. If the client receives a final answer without an approval request, inspect the saved tool policy and whether retrieval was invoked.

## 3. Set up the Python client

The current lab specifies Python **3.13**. It was tested with 3.13.12 and does not support Python 3.14 in its dependency set.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/04-integrate-agent-with-foundry-iq/Python
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

Create `.env` with the project endpoint and the saved agent's name:

```dotenv
PROJECT_ENDPOINT=https://<foundry-resource>.services.ai.azure.com/api/projects/<project-name>
AGENT_NAME=product-expert-agent
```

The client needs Foundry project access through your signed-in identity. Search authentication is already stored in the portal connection; there is no Search key in this client. Do not put a Search admin key into a prompt or commit it to source control.

## 4. Complete `agent_client.py`

This merges the instruction-page additions into a compact executable client. It retains approval and denial, supports multiple requests and tool rounds, displays returned citation metadata, and cleans up the conversation it creates. The `history` command shows a local transcript; the service-side conversation is what maintains agent context.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def ask_agent(client, conversation_id, reference, question):
    client.conversations.items.create(
        conversation_id=conversation_id,
        items=[{"type": "message", "role": "user", "content": question}],
    )
    response = client.responses.create(
        conversation=conversation_id, input="", extra_body=reference,
    )
    for round_number in range(9):
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        requests = [item for item in response.output
                    if item.type == "mcp_approval_request"]
        if not requests:
            return response
        if round_number == 8:
            raise RuntimeError("Stopped after eight MCP approval rounds.")
        decisions = []
        for request in requests:
            print(f"\nServer: {request.server_label}\nTool: {request.name}")
            print(f"Arguments: {request.arguments}")
            approved = input("Approve this knowledge operation? [y/N] ").lower() == "y"
            decisions.append({"type": "mcp_approval_response",
                              "approval_request_id": request.id, "approve": approved})
        client.conversations.items.create(conversation_id=conversation_id, items=decisions)
        response = client.responses.create(
            conversation=conversation_id, input="", extra_body=reference,
        )
    raise RuntimeError("The tool loop did not complete.")


def display_response(response):
    print(f"\nAgent: {response.output_text}")
    # Citation metadata lives on message text annotations, when supplied.
    seen = set()
    for item in response.output:
        if item.type != "message":
            continue
        for part in item.content:
            if part.type != "output_text":
                continue
            for annotation in part.annotations or []:
                source = getattr(annotation, "url", None) or getattr(annotation, "file_id", None)
                if source and source not in seen:
                    title = getattr(annotation, "title", None) or getattr(annotation, "filename", "Source")
                    print(f"  {title}: {source}")
                    seen.add(source)


def main():
    load_dotenv()
    # Match the local-development lab: use the signed-in developer credential,
    # rather than an unrelated environment or managed-identity credential.
    with (
        DefaultAzureCredential(exclude_environment_credential=True,
                               exclude_managed_identity_credential=True) as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.get(agent_name=os.environ["AGENT_NAME"])
        reference = {"agent_reference": {"name": agent.name, "type": "agent_reference"}}
        conversation = client.conversations.create(items=[])
        history = []
        try:
            while True:
                question = input("You (history / exit): ").strip()
                if question.lower() in {"exit", "quit"}:
                    break
                if question.lower() == "history":
                    for role, text in history:
                        print(f"{role}: {text}")
                    continue
                if not question:
                    continue
                response = ask_agent(client, conversation.id, reference, question)
                history.extend([("You", question), ("Agent", response.output_text)])
                display_response(response)
        finally:
            client.conversations.delete(conversation_id=conversation.id)


if __name__ == "__main__":
    main()
```

The credential exclusions above are specific to this interactive developer lab. An application deployed in Azure should use the credential appropriate to its hosting environment, commonly a managed identity, with the required roles.

## 5. Run and test retrieval

```bash
python agent_client.py
```

| Test | What to inspect |
| --- | --- |
| `What tents does Contoso sell?` | A knowledge retrieval request, relevant product facts, and citations you can trace to the PDFs |
| `Which extra-large backpacks are available?` | Matching catalog details rather than a generic camping answer |
| `Compare a tent and a backpack for a multi-day hike. Include useful accessories.` | Evidence combined across relevant documents |
| `Does Contoso sell smartphones?` | A clear statement when the knowledge base has no relevant information |
| Deny a retrieval request | The client submits `approve=False`; the answer should explain the limitation without inventing catalog evidence |

Inspect retrieved evidence and citation destinations, not only whether the answer contains a citation-looking string. RAG reduces reliance on model memory but does not guarantee that every answer is correct or every generated citation is supported. If no results appear, check source ingestion status, Search connection authentication, the agent's saved knowledge tool, and the contents of the uploaded PDFs.

## Cleanup and attribution

`exit` deletes this client's conversation. It preserves the agent and knowledge base you configured in the portal. When finished, remove the dedicated lab's Foundry, Search, and Storage resources, or delete the dedicated resource group after confirming it holds only your lab resources. Removing only the Python client does not stop resource charges.

Adapted from MicrosoftLearning instructions and starter code under the **MIT License**; see [Third-party notices](../../../THIRD_PARTY_NOTICES.md). Syntax and SDK usage were checked; provisioning, retrieval quality, permissions, and billing behavior require a run in your own Azure environment.
