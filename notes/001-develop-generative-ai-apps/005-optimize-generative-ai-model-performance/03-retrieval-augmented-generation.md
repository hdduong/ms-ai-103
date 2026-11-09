# 03 Ground your model with Retrieval Augmented Generation

[Ground your model with Retrieval Augmented Generation](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/3-retrieval-augmented-generation?pivots=text)

- An ungrounded model only knows its training data and can invent names and prices. Grounding puts trusted text in the prompt.
- RAG steps: **retrieve**, **augment** the prompt, **generate**.
- An embedding is a vector of the meaning. Similar sentences sit close together. Cosine similarity near 1 means a close match.
- Azure AI Search is the index. Load data from Blob, Data Lake Gen2, OneLake, or an upload. The embedding model fills the index.
- Search combines keyword or vector retrieval with options such as semantic ranking. Learn recommends **hybrid** for generative apps: it merges keyword and vector results. [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview) explains why exact identifiers favor keyword matches while conceptual matches favor vectors; evaluate the combination on your documents.
- Use RAG for private data, data that changes, facts that must be right, and anything after the training cutoff.
- Agents that should not run their own search can use **Foundry IQ**.

## Connect the retrieval steps to the travel application

In [the file-search application](../004-use-generative-ai-tools/05-file-search.md#lab-brochures-plus-web-search), the service searches the uploaded brochures, adds matching text to the model's context, and generates an answer. The application supplies the store ID and question, then inspects tool results and citations. It does not separately compute embeddings or create an Azure AI Search index.

The following snippet exposes the **generation** step for a custom retrieval pipeline. `openai_client` comes from [the endpoint lesson](../003-foundry-sdk/03-choose-an-endpoint-and-sdk.md); `retrieved_context` must be the text your retrieval step already returned. It is a continuation, not a complete search program.

```python
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    input=[
        {
            "role": "system",
            "content": "You are a helpful travel advisor. Use the following hotel data to answer: " + retrieved_context,
        },
        {"role": "user", "content": "Which hotels do you offer in Paris?"},
    ],
)
print(response.output_text)
```
