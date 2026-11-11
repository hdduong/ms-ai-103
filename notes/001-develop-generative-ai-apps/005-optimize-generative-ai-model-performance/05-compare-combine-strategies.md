# 05 Compare and combine optimization strategies

[Compare and combine optimization strategies](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/5-compare-combine-strategies?pivots=text)

- Prompt engineering is the base. **RAG** adds facts the model does not have. **Fine-tuning** makes style and format consistent.
- Prompt engineering: low setup cost and no additional training. Instructions alone do not retrieve unknown facts; supplied context can contain private or current facts. Long prompts consume more tokens.
- RAG: medium cost (search, storage, tokens). Quality depends on the index and the chunks.
- Fine-tuning: highest cost (training, hosting, tokens). Most consistent behavior, and prompts can be shorter. Retrain when the base model or the requirements change.

| Need | Add |
| --- | --- |
| Tone, format, instructions | Prompt engineering |
| Private, current, or factual data | RAG |
| More consistent voice or format across many requests | Fine-tuning, followed by evaluation |

- Common stacks: prompt + RAG (how to act, and what to know), prompt + fine-tune (style in the weights, session rules in the system message), RAG + fine-tune, or all three.
- Order: prompt, then RAG if accuracy needs data, then fine-tune if the style still slips.

## Prompt plus RAG in one call

The [travel application](../004-use-generative-ai-tools/05-file-search.md#lab-brochures-plus-web-search) already combines these choices. The following is its generation step, after `client` and `vector_store` have been created and the brochures have finished indexing. Reuse that setup; this snippet is not a separate program or another upload step.

```python
response = client.responses.create(
    model=os.environ["MODEL_DEPLOYMENT"],  # deployment name
    instructions=(
        "You are a travel assistant for Margie's Travel. Answer about its "
        "services from the brochures. Use the web for general destination advice."
    ),
    input="What hotels does Margie's Travel offer in San Francisco?",
    tools=[
        {"type": "file_search", "vector_store_ids": [vector_store.id]},
        {"type": "web_search"},
    ],
)
print(response.output_text)
```

`instructions` is prompt engineering: it defines the role and source-selection rule. `file_search` retrieves company facts; `web_search` retrieves public information. Neither tool changes model weights. If facts are wrong, inspect retrieval and source data before deciding to fine-tune.

The [fine-tuning lesson](04-fine-tune-model.md#lab-fine-tune-gpt-5-in-the-portal) uses a **different** travel-assistant scenario: its dataset discourages hotel and restaurant recommendations. It teaches behavior adaptation, so do not attach that dataset to the brochure agent and expect it to preserve the brochure agent's goals. Evaluate a consistent scenario before combining all three techniques.
