# 05 Compare and combine optimization strategies

[Compare and combine optimization strategies](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/5-compare-combine-strategies?pivots=text)

- Prompt engineering is the base. **RAG** adds facts the model does not have. **Fine-tuning** makes style and format consistent.
- Prompt engineering: fastest and cheapest, per-token only. It cannot add unknown facts, and long prompts cost more.
- RAG: medium cost (search, storage, tokens). Quality depends on the index and the chunks.
- Fine-tuning: highest cost (training, hosting, tokens). Most consistent behavior, and prompts can be shorter. Retrain when the base model or the requirements change.

| Need | Add |
| --- | --- |
| Tone, format, instructions | Prompt engineering |
| Private, current, or factual data | RAG |
| The same voice every time | Fine-tuning |

- Common stacks: prompt + RAG (how to act, and what to know), prompt + fine-tune (style in the weights, session rules in the system message), RAG + fine-tune, or all three.
- Order: prompt, then RAG if accuracy needs data, then fine-tune if the style still slips.

## Prompt plus RAG in one call

From the [tools lab](../004-use-generative-ai-tools/05-file-search.md#lab-brochures-plus-web-search). `instructions` is the prompt engineering. `file_search` over the brochures is the RAG. `web_search` covers what the brochures do not.

```python
import glob
from contextlib import ExitStack

vector_store = openai_client.vector_stores.create(
    name="travel-brochures",
)
with ExitStack() as stack:
    file_streams = [
        stack.enter_context(open(path, "rb"))
        for path in glob.glob("brochures/*.pdf")
    ]
    openai_client.vector_stores.file_batches.upload_and_poll(
        vector_store_id=vector_store.id,
        files=file_streams,
    )

response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are a travel assistant for Margie's Travel. Answer about its services from the brochures. Use the web for general destination advice.",
    input="What hotels does Margie's Travel offer in San Francisco?",
    tools=[
        {"type": "file_search", "vector_store_ids": [vector_store.id]},
        {"type": "web_search"},
    ],
)
print(response.output_text)
```

The fine-tune side, with the lab's JSONL and job settings, is in [04](04-fine-tune-model.md#lab-fine-tune-gpt-5-in-the-portal).
