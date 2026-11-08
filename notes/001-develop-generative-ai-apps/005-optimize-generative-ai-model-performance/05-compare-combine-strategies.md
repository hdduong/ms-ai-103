# 05 Compare and combine optimization strategies

[Compare and combine optimization strategies](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/5-compare-combine-strategies?pivots=text)

- Prompt engineering is the base. **RAG** adds facts the model does not have. **Fine-tuning** locks style and format.
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

From the tools lab, [Create a generative AI app that uses tools](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04a-use-own-data.html). `instructions` is the prompt engineering. `file_search` over the brochures is the RAG. `web_search` covers what the brochures do not.

```python
import glob

vector_store = openai_client.vector_stores.create(
    name="travel-brochures",
)
file_streams = [open(path, "rb") for path in glob.glob("brochures/*.pdf")]
openai_client.vector_stores.file_batches.upload_and_poll(
    vector_store_id=vector_store.id,
    files=file_streams,
)
for stream in file_streams:
    stream.close()

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

The fine-tune exercise trains on JSONL. Each line is one conversation: the same system message, a user prompt, and the reply style you want.

```json
{"messages": [
  {"role": "system", "content": "You are an AI travel assistant. Do not recommend hotels, flights, cars, or restaurants."},
  {"role": "user", "content": "What's a must-see in Paris?"},
  {"role": "assistant", "content": "Start at the Eiffel Tower, then the Louvre. What kind of attractions do you want?"}
]}
```
