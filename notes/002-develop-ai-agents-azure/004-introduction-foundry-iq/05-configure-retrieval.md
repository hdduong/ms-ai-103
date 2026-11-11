# 05 Configure retrieval with Foundry IQ

[Configure retrieval with Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/5-configure-retrieval?pivots=text)

- A vague instruction ("use the knowledge base") lets the model answer from training data or skip the citation.
- Say three things: always search first, the citation format, and what to say when the base has no answer.
- Test four prompts: a fact, a question that needs several documents, a question outside the base, and an ambiguous one.
- A good reply is grounded, cited, relevant, and complete.
- Support agents should refuse to guess. Research agents may combine sources and should cite all of them. Compliance agents stay inside the policy text and name the section and effective date.
- In production, watch citation rate, how often it says it does not know, and whether the retrieved docs actually answer the question.
- The Learn unit shows `extra_body["agent"]`; the exercise and code here use `extra_body["agent_reference"]` with `type="agent_reference"`. Follow the exercise syntax consistently. The saved agent already defines its model, so this call has no `model` argument.

## Instructions from the exercise

The lab sets these instructions on the agent, then adds Foundry IQ under Knowledge. The client does not pass `model`. It passes `agent_reference`.

![product-expert-agent instructions: always search the knowledge base for product questions and cite sources.](images/05-agent-instructions.png)

The knowledge base has its own retrieval settings: the chat model that plans queries, **Retrieval reasoning effort**, and **Retrieval instructions** that say which source to try first.

![knowledgebase301 settings: gpt-5 as the planning model, retrieval reasoning effort Minimal, retrieval instructions, and the knowledge sources.](images/05-knowledge-base-config.png)

```text
You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products.
You must ALWAYS search the knowledge base to answer questions about our products or product catalog.
Provide detailed, accurate information and always cite your sources.
If you don't find relevant information in the knowledge base, say so clearly.
```

## Lab: ask, then approve the knowledge-base call

From the [Foundry IQ lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04-integrate-agent-with-foundry-iq.html). Set the instructions above on `product-expert-agent` first. The portal connection may auto-approve, so in VS Code open the agent, find the tool named `kb-knowledgebase...`, **Configure > Never auto-approve tools**, and save.

Use Python 3.13 as specified by the exercise, and create a working directory for the client:

```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

`.env`:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
AGENT_NAME=product-expert-agent
```

### Resume the agent after a decision

Save the next two Python blocks together as `agent_client.py`. The first block implements one question and its possible approval rounds. It adds the user message, asks the saved agent to respond, then appends approval decisions to the same conversation.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv


def answer_question(client, conversation_id, reference, question):
    client.conversations.items.create(
        conversation_id=conversation_id,
        items=[{"type": "message", "role": "user", "content": question}],
    )
    for round_number in range(9):
        response = client.responses.create(
            conversation=conversation_id, extra_body=reference, input="",
        )
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        requests = [item for item in response.output if item.type == "mcp_approval_request"]
        if not requests:
            return response.output_text
        if round_number == 8:
            raise RuntimeError("Stopped after eight knowledge-base approval rounds.")
        decisions = []
        for request in requests:
            print(f"Server: {request.server_label}  Tool: {request.name}")
            print("Arguments:", request.arguments)
            decisions.append({
                "type": "mcp_approval_response",
                "approval_request_id": request.id,
                "approve": input("Approve? [y/N] ").strip().lower() == "y",
            })
        client.conversations.items.create(
            conversation_id=conversation_id, items=decisions,
        )
    raise RuntimeError("Stopped after eight knowledge-base approval rounds.")
```

The empty `input` is deliberate: this pattern already put the question or decision into the conversation. `approval_request_id` connects the decision to the pending lookup. The next response either requests another lookup or returns the final answer. The exercise approves automatically; the prompt above is an adaptation that lets you inspect and deny the requested operation.

### Connect the client to the saved portal agent

Append this block to `agent_client.py`:

```python
def main():
    load_dotenv()
    with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True,
        ) as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.get(agent_name=os.environ["AGENT_NAME"])
        reference = {"agent_reference": {"name": agent.name, "type": "agent_reference"}}
        conversation = client.conversations.create(items=[])
        try:
            while True:
                question = input("You (exit to stop): ").strip()
                if question.lower() in {"exit", "quit"}:
                    break
                if question:
                    print("Agent:", answer_question(client, conversation.id, reference, question))
        finally:
            client.conversations.delete(conversation_id=conversation.id)


if __name__ == "__main__":
    main()
```

The credential exclusions follow the exercise's local developer setup so environment or managed-identity credentials do not override the signed-in developer. The client authenticates to Foundry; Search authentication remains in the project connection configured in [unit 04](04-data-requirements.md#lab-blob-source-for-product-expert-agent). Closing the program deletes its conversation and closes the clients, while keeping the portal-created agent for further study.

```bash
python agent_client.py
```

### Check grounding, citations, and refusal

| Ask | Expect |
| --- | --- |
| `What tents does Contoso sell?` | A knowledge-base call, product facts, and citations to the PDFs |
| `Which extra-large backpacks are available?` | Catalog details, not a generic camping answer |
| `Compare a tent and a backpack for a multi-day hike. Include useful accessories.` | Facts from more than one document |
| `Does Contoso sell smartphones?` | A clear "not in the knowledge base" |
| Deny the approval | An answer that says it could not search, with no invented products |
