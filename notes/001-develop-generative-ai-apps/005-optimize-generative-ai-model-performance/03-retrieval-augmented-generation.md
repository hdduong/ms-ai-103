# 03 Ground your model with Retrieval Augmented Generation

[Ground your model with Retrieval Augmented Generation](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/3-retrieval-augmented-generation?pivots=text)

- An ungrounded model only knows its training data and can invent names and prices. Grounding puts trusted text in the prompt.
- RAG steps: **retrieve**, **augment** the prompt, **generate**.
- An embedding is a vector of the meaning. Similar sentences sit close together. Cosine similarity near 1 means a close match.
- Azure AI Search is the index. Load data from Blob, Data Lake Gen2, OneLake, or an upload. The embedding model fills the index.
- Search types: keyword, semantic, vector, **hybrid**. Hybrid is the one to use for generative apps.
- Use RAG for private data, data that changes, facts that must be right, and anything after the training cutoff.
- Agents that should not run their own search can use **Foundry IQ**.

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
