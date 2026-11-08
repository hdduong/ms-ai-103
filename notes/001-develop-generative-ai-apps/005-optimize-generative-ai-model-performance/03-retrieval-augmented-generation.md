# 03 Ground your model with Retrieval Augmented Generation

[Ground your model with Retrieval Augmented Generation](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/3-retrieval-augmented-generation?pivots=text)

- An ungrounded model only knows its training data and can invent names and prices. Grounding puts trusted text in the prompt.
- RAG steps: **retrieve**, **augment** the prompt, **generate**.
- An embedding is a vector of the meaning. Similar sentences sit close together. Cosine similarity near 1 means a close match.
- Azure AI Search is the index. Load data from Blob, Data Lake Gen2, OneLake, or an upload. The embedding model fills the index.
- Retrieval options include keyword, vector, and **hybrid** (keyword + vector). Semantic ranking can rerank results. Hybrid is a useful candidate, not a universal requirement: evaluate on your questions and documents. See [Azure AI Search hybrid search](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview).
- Use RAG for private data, data that changes, facts that must be right, and anything after the training cutoff.
- Agents that should not run their own search can use **Foundry IQ**.

This fragment is the **generation step only**: `retrieved_context` must already contain text returned by your retrieval pipeline, and `openai_client` must be configured. For an end-to-end managed retrieval example with supplied PDFs, use the [file-search exercise](../004-use-generative-ai-tools/07-exercise-use-tools.md).

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
