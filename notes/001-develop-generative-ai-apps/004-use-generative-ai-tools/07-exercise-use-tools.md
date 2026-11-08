# 07 Exercise: Create a generative AI app that uses tools

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/07-exercise)

> Reviewed **2026-11-07**. The official exercise implements **web search and file search** for Margie's Travel. Code interpreter and custom functions are covered in the module's separate units; they are not additional steps in this lab. Allow about **30 minutes**.

## 1. Set up the lab

Use an Azure subscription, New Foundry project, deployed **gpt-5.2**, Azure CLI, Git, VS Code, and Python **3.13.x** (the lab tested **3.13.12**). Copy the **Azure OpenAI endpoint**, including the `/openai/v1/` base path, and the deployment name.

First try a travel question in the playground. Add **web_search**, then repeat the question to compare answers with external information available.

To use the reviewed lab snapshot on macOS/Linux:

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-studio.git
cd mslearn-ai-studio
git checkout b702dc2fd9f9411b5eb1f86b9259db6515cd9c48
cd labfiles/tools/python/tools-app
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

If you already cloned that snapshot for the chat lab, navigate to the tools directory instead of cloning again. On Windows use `py -3.13 -m venv .venv` and `.venv\Scripts\Activate.ps1`. The pinned requirements are:

```text
python-dotenv
azure-identity
openai
```

They are unpinned package names, not a reproducible dependency lock. Keep this environment separate from labs using other SDK families, and check the current official requirements before changing dependency versions.

Create `.env` in this directory:

```dotenv
AZURE_OPENAI_ENDPOINT="https://<resource-name>.openai.azure.com/openai/v1/"
MODEL_DEPLOYMENT="<your-gpt-5.2-deployment-name>"
```

Keep the supplied `brochures/` folder beside `tools-app.py`. It contains the lab's PDFs for Margie's Travel and its destinations. The signed-in Azure identity needs permission to call the deployment and use its tools.

## 2. Complete the tools application

The upstream file contains TODOs. This assembled version follows the exercise and adds `ExitStack` so local file handles close even if upload fails, checks for PDFs before creating a store, and closes SDK clients.

```python
import glob
import os
from contextlib import ExitStack

from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import OpenAI


def main():
    load_dotenv()
    endpoint = os.environ["AZURE_OPENAI_ENDPOINT"]
    deployment = os.environ["MODEL_DEPLOYMENT"]
    pdf_paths = sorted(glob.glob("brochures/*.pdf"))
    if not pdf_paths:
        raise FileNotFoundError("Run from the lab directory containing brochures/*.pdf")

    instructions = (
        "You are a travel assistant that provides information on travel services "
        "available from Margie's Travel. Answer questions about services offered "
        "by Margie's Travel using the provided travel brochures. Search the web "
        "for general information about destinations or current travel advice."
    )

    with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        with OpenAI(base_url=endpoint, api_key=token_provider) as client:
            vector_store = client.vector_stores.create(name="travel-brochures")
            print("Vector store:", vector_store.id)
            with ExitStack() as stack:
                streams = [stack.enter_context(open(path, "rb")) for path in pdf_paths]
                batch = client.vector_stores.file_batches.upload_and_poll(
                    vector_store_id=vector_store.id,
                    files=streams,
                )
            print("Files indexed:", batch.file_counts.completed)
            if batch.file_counts.failed:
                raise RuntimeError("Some brochures failed to index; inspect the file batch.")

            last_response_id = None
            while True:
                prompt = input("Enter a travel question (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue

                response = client.responses.create(
                    model=deployment,
                    instructions=instructions,
                    input=prompt,
                    previous_response_id=last_response_id,
                    tools=[
                        {"type": "file_search", "vector_store_ids": [vector_store.id]},
                        {"type": "web_search"},
                    ],
                )
                print(response.output_text)
                last_response_id = response.id


if __name__ == "__main__":
    main()
```

Save it as `tools-app.py` in the lab directory and run:

```bash
python tools-app.py
```

## 3. Test retrieval and conversation context

1. Ask for general advice about visiting **San Francisco next month**.
2. Follow up by asking which hotels **Margie's Travel** offers there.
3. Inspect the response's sources and the tool calls in `response.output` when checking which tool actually ran. Merely declaring a tool does not prove it was used.
4. Enter `quit` to exit.

The first question can benefit from current web information. The second should be grounded in the company brochures. Instructions guide the model's choice; verify actual behavior rather than assuming a particular prompt always selects one tool.

## What the code teaches

| Code | Why it is there |
| --- | --- |
| `vector_stores.create` | Creates the searchable store for the brochures |
| `upload_and_poll` | Uploads and waits for file processing before asking grounded questions |
| `file_search` + `vector_store_ids` | Restricts the file search source to the chosen store |
| `web_search` | Makes web retrieval available for public/current information |
| `instructions` | Describes which information source fits each type of question |
| `previous_response_id` | Preserves conversational context for follow-up questions |

This is retrieval at inference time: uploading brochures does **not** fine-tune the model. File search is a managed retrieval option; a custom Azure AI Search pipeline offers different control over retrieval and ranking.

For the other tool patterns, see [Code interpreter](03-code-interpreter.md) and [Function calling](06-function.md).

## Resource lifecycle

The lab creates a new vector store and uploads files on each run. Record the printed store ID, and reuse an existing store in a longer-lived application instead of repeatedly ingesting the same documents. Exiting the Python process closes local handles and clients; it does not delete cloud files or stores. Remove the exercise's disposable resources when finished, using the official lab cleanup guidance.

## Source and attribution

Source: Microsoft's [Create a generative AI app that uses tools](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/04a-use-own-data.md), [starter application, requirements, and brochures](https://github.com/MicrosoftLearning/mslearn-ai-studio/tree/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/labfiles/tools/python/tools-app), commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. The code is assembled/adapted from the exercise with file/client cleanup, explicit configuration lookup, a failed-indexing check, and blank-input handling. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md).

Validation during this review: Python syntax, source comparison, and the supplied asset paths. No authenticated Azure tool calls or paid lab runs were executed.
