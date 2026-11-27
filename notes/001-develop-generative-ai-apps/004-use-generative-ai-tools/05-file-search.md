# 05 Use the file_search tool

[Use the file_search tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/05-file-search?pivots=text)

- Grounds the answer in files you upload (policy, manuals, contracts). Retrieval is by meaning, not only keywords.
- Create a vector store, upload the file, then pass `vector_store_ids`. `include=["file_search_call.results"]` returns the matched chunks.
- Split stores by domain. Re-index after the file changes. A huge mixed store retrieves worse.
- For many data stores behind one agent, the unit points at **Foundry IQ** instead of this tool.

```python
# Empty store that will index the policy file for search
vector_store = openai_client.vector_stores.create(name="policy-docs")
# Upload the PDF into the store and wait until it is processed
with open("expenses_policy.pdf", "rb") as policy_file:
    openai_client.vector_stores.files.upload_and_poll(
        vector_store_id=vector_store.id,
        file=policy_file,
    )
# Ask; the model can search that store before it answers
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are an AI assistant that provides information from HR policy documents.",
    input="What's the maximum amount I can claim for a taxi ride?",
    tools=[
        {"type": "file_search", "vector_store_ids": [vector_store.id]},
    ],
    include=["file_search_call.results"],  # also return the matched chunks
)
print(response.output_text)
```

## Lab: brochures plus web search

The [Create a generative AI app that uses tools](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04a-use-own-data.html) exercise applies the same retrieval pattern to **Margie's Travel**. Company offerings belong in the brochures; current destination information comes from [web search](04-web-search.md). The model receives both tools and instructions explaining when each is useful.

### Prepare the documents and connection

Use Python 3.13 and the deployed `gpt-5.2` model, Azure sign-in, and `.env` from [the chat lesson](../003-foundry-sdk/04-responses-api.md#prepare-the-client). Get the [official starter folder and brochures](https://github.com/MicrosoftLearning/mslearn-ai-studio/tree/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/labfiles/tools/python/tools-app). If you already cloned `mslearn-ai-studio`, open `labfiles/tools/python/tools-app`; otherwise clone that repository first. Keep its `brochures/` PDFs beside `tools-app.py`, with `.env` in the same folder.

In your activated virtual environment:

```bash
# Install the lab packages
python -m pip install python-dotenv azure-identity openai
# Sign in so DefaultAzureCredential can use your account
az login
```

### Index first, then ask a grounded question

Save this assembled program as `tools-app.py`. The code follows the exercise and adds file/client cleanup plus checks for missing PDFs and failed indexing.

```python
import os
from contextlib import ExitStack
from pathlib import Path

from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import OpenAI


def main():
    # Paths start at this script's folder, so it runs from anywhere
    app_folder = Path(__file__).resolve().parent
    load_dotenv(app_folder / ".env")
    # Stop early if the brochure PDFs are missing
    pdf_paths = sorted((app_folder / "brochures").glob("*.pdf"))
    if not pdf_paths:
        raise FileNotFoundError("Put the lab's brochures/*.pdf beside tools-app.py")

    # A role, then a source rule: brochures for offerings, web for advice
    instructions = (
        "You are a travel assistant that provides information on travel services "
        "available from Margie's Travel. Answer questions about services offered "
        "by Margie's Travel using the provided travel brochures. Search the web "
        "for general information about destinations or current travel advice."
    )
    # Sign in with az login and connect to the Azure OpenAI endpoint
    with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        with OpenAI(
            base_url=os.environ["AZURE_OPENAI_ENDPOINT"],
            api_key=token_provider,
        ) as client:
            # A searchable store for the brochures; print its id
            vector_store = client.vector_stores.create(name="travel-brochures")
            print("Vector store:", vector_store.id)
            # Open every PDF; ExitStack closes them all when the block ends
            with ExitStack() as stack:
                file_streams = [
                    stack.enter_context(path.open("rb")) for path in pdf_paths
                ]
                # Upload all PDFs; wait for processing before any question
                batch = client.vector_stores.file_batches.upload_and_poll(
                    vector_store_id=vector_store.id,
                    files=file_streams,
                )
            print("Files indexed:", batch.file_counts.completed)
            # Stop if any brochure failed; its content would be unsearchable
            if batch.file_counts.failed:
                raise RuntimeError("A brochure failed to index; inspect the file batch.")

            last_response_id = None
            # One question per pass until you type quit
            while True:
                prompt = input("Enter a travel question (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue
                # Offer both tools; the instructions guide which one to use
                response = client.responses.create(
                    model=os.environ["MODEL_DEPLOYMENT"],  # deployment name
                    instructions=instructions,
                    input=prompt,
                    previous_response_id=last_response_id,  # lets follow-ups refer back
                    tools=[
                        # Lets the model search the brochure store
                        {"type": "file_search", "vector_store_ids": [vector_store.id]},
                        # Second source: public and current information
                        {"type": "web_search"},
                    ],
                    include=["file_search_call.results"],  # add retrieved chunks to inspect
                )
                print(response.output_text)
                # Offering a tool doesn't mean it ran; print the tools used
                for item in response.output:
                    if item.type in {"file_search_call", "web_search_call"}:
                        print("Tool used:", item.type)
                # The next question continues from this turn
                last_response_id = response.id


if __name__ == "__main__":
    main()
```

Run `python tools-app.py`. First ask about visiting San Francisco next month, then ask which hotels Margie's Travel offers there. Check the printed tool types, source citations, and retrieved results against the PDFs. The first question may use the web; the second should use the brochures. Treat these as behaviors to verify, since the model chooses its calls.

Uploading documents supports retrieval during inference; it does not train the model. `ExitStack` closes local PDF handles, and the context managers close clients. They do **not** delete Azure files or the vector store. This teaching program creates a store on each run: save the printed ID for inspection, reuse a store in a longer-lived application, and remove the disposable lab resources when finished.
