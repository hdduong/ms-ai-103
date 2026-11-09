# 05 Use the file_search tool

[Use the file_search tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/05-file-search?pivots=text)

- Grounds the answer in files you upload (policy, manuals, contracts). Retrieval is by meaning, not only keywords.
- Create a vector store, upload the file, then pass `vector_store_ids`. `include=["file_search_call.results"]` returns the matched chunks.
- Split stores by domain. Re-index after the file changes. A huge mixed store retrieves worse.
- For many data stores behind one agent, the unit points at **Foundry IQ** instead of this tool.

```python
vector_store = openai_client.vector_stores.create(name="policy-docs")
with open("expenses_policy.pdf", "rb") as policy_file:
    openai_client.vector_stores.files.upload_and_poll(
        vector_store_id=vector_store.id,
        file=policy_file,
    )
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are an AI assistant that provides information from HR policy documents.",
    input="What's the maximum amount I can claim for a taxi ride?",
    tools=[
        {"type": "file_search", "vector_store_ids": [vector_store.id]},
    ],
    include=["file_search_call.results"],
)
print(response.output_text)
```

## Lab: brochures plus web search

Lab: [Create a generative AI app that uses tools](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04a-use-own-data.html). Same `.env` as the [chat lab](../003-foundry-sdk/04-responses-api.md#lab-chat-app). `pip install python-dotenv azure-identity openai`. The `brochures/` folder of PDFs sits next to `tools-app.py`.

```python
import glob
import os
from contextlib import ExitStack
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(),
    "https://ai.azure.com/.default",
)
client = OpenAI(
    base_url=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=token_provider,
)

vector_store = client.vector_stores.create(name="travel-brochures")
with ExitStack() as stack:
    file_streams = [
        stack.enter_context(open(path, "rb"))
        for path in glob.glob("brochures/*.pdf")
    ]
    batch = client.vector_stores.file_batches.upload_and_poll(
        vector_store_id=vector_store.id,
        files=file_streams,
    )
print("Files indexed:", batch.file_counts.completed)

instructions = (
    "You are a travel assistant that provides information on travel services "
    "available from Margie's Travel. Answer questions about services offered "
    "by Margie's Travel using the provided travel brochures. Search the web "
    "for general information about destinations or current travel advice."
)
last_response_id = None
while True:
    prompt = input("Enter a travel question (or 'quit' to exit): ").strip()
    if prompt.lower() == "quit":
        break
    response = client.responses.create(
        model=os.environ["MODEL_DEPLOYMENT"],  # deployment name
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
```

- Ask about visiting San Francisco next month (web), then which hotels Margie's Travel offers there (brochures). Look at the tool calls in `response.output` to see which tool ran.
- `upload_and_poll` waits until the files are indexed. Uploading brochures is retrieval at run time, not fine-tuning.
- Each run creates a new vector store. A real app reuses the store id.
