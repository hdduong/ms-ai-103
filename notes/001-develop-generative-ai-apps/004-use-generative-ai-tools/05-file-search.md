# 05 Use the file_search tool

[Use the file_search tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/05-file-search?pivots=text)

- Grounds the answer in files you upload (policy, manuals, contracts). Retrieval is by meaning, not only keywords.
- Create a vector store, upload the file, then pass `vector_store_ids`. `include=["file_search_call.results"]` returns the matched chunks.
- Split stores by domain. Re-index after the file changes. A huge mixed store retrieves worse.
- For many data stores behind one agent, the unit points at **Foundry IQ** instead of this tool.

```python
vector_store = openai_client.vector_stores.create(name="policy-docs")
openai_client.vector_stores.files.upload_and_poll(
    vector_store_id=vector_store.id,
    file=open("expenses_policy.pdf", "rb"),
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
