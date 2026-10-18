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
