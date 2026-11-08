# 02 Optimize model output with prompt engineering

[Optimize model output with prompt engineering](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/2-prompt-engineering?pivots=text)

- First optimization. No extra training or index. A prompt still does not guarantee the model obeys it.
- Pieces: **system** (role, tone, format, limits), **user**, **assistant** (prior replies), and **examples**. In the playground the system text is the **Instructions** box.

![Playground instructions for gpt-4.1. That box is the system message.](../003-foundry-sdk/images/02-playground.png)
- System-message checklist: role, boundaries, output format, and what to do when unsure.
- Patterns: persona, a format template, chain of thought (for non-reasoning models; reasoning models already do the steps), few-shot versus zero-shot.
- Separate instructions from source text with `---`, headings, or XML tags. If instructions are ignored, repeat them at the end. Later text weighs more (recency bias).
- Where the model supports them, adjust **temperature** or **top_p** one at a time. Lower temperature reduces sampling variation; it does not make an answer factually correct. Check [reasoning-model parameter support](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/reasoning) before sending these settings.
- Start here for tone, format, and instructions. You can include private or recent facts as prompt context, but prompting alone does not retrieve them automatically. Use RAG when that context must be found from a data source; long prompts also cost tokens.
