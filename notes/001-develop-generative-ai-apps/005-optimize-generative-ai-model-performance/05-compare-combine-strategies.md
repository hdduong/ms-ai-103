# 05 Compare and combine optimization strategies

[Compare and combine optimization strategies](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/5-compare-combine-strategies?pivots=text)

- Prompt engineering is the base. **RAG** adds facts the model does not have. **Fine-tuning** can improve consistency of style and format; it does not guarantee compliance.
- Prompt engineering: fastest and cheapest, per-token only. It can include supplied facts as context but does not automatically retrieve them, and long prompts cost more.
- RAG: medium cost (search, storage, tokens). Quality depends on the index and the chunks.
- Fine-tuning: highest cost (training, hosting, tokens). Most consistent behavior, and prompts can be shorter. Retrain when the base model or the requirements change.

| Need | Add |
| --- | --- |
| Tone, format, instructions | Prompt engineering |
| Private, current, or factual data | RAG |
| More consistent voice across representative tests | Fine-tuning |

- Common stacks: prompt + RAG (how to act, and what to know), prompt + fine-tune (style in the weights, session rules in the system message), RAG + fine-tune, or all three.
- Order: prompt, then RAG if accuracy needs data, then fine-tune if the style still slips.

## Prompt plus RAG in one call

Adapted from the tools lab, [Create a generative AI app that uses tools](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/04a-use-own-data.md). See the [complete exercise lesson](../004-use-generative-ai-tools/07-exercise-use-tools.md) for setup and the current `gpt-5.2` deployment. The fragment below retains the earlier `gpt-4.1` example and assumes an initialized `openai_client`. `instructions` is the prompt engineering. `file_search` over the brochures is the RAG. `web_search` covers what the brochures do not.

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

## Fine-tune training data

The [fine-tune exercise](06-exercise-fine-tune.md) trains on JSONL. Each physical line is one conversation: the system message, a user prompt, and the desired reply. The simplified example below is expanded for reading; put each complete object on one line in the actual `.jsonl` file. See [third-party notices](../../../THIRD_PARTY_NOTICES.md) for the adapted exercise code/data.

```json
{"messages": [
  {"role": "system", "content": "You are an AI travel assistant. Do not recommend hotels, flights, cars, or restaurants."},
  {"role": "user", "content": "What's a must-see in Paris?"},
  {"role": "assistant", "content": "Start at the Eiffel Tower, then the Louvre. What kind of attractions do you want?"}
]}
```
