# 04 Fine-tune a model for consistent behavior

[Fine-tune a model for consistent behavior](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/4-fine-tune-model?pivots=text)

- Fine-tuning changes weights so style, tone, or format stay consistent when prompts and few-shot examples do not.
- **LoRA** is the parameter-efficient technique introduced by this unit; it learns a smaller set of adapter parameters. Do not assume every model/method exposes the same training implementation; check the selected model's supported fine-tuning methods.
- Measure a baseline first. Without one you cannot tell if the fine-tune helped.
- Use it for brand voice, a schema the model keeps missing, shorter prompts, distillation onto a smaller model, or better tool selection.
- Methods: **SFT** (prompt and response pairs), **RFT** (a grader rewards better answers), **DPO** (preferred versus rejected pairs). SFT then DPO is a valid sequence. The portal labels SFT as **Supervised**.

![Build > Fine-tune. A completed Supervised job on gpt-4.1.](images/04-finetune.png)
- Training file is **JSONL**. Each line is `messages` with system, user, and assistant. Keep the same system message in training and at inference. The unit recommends hundreds of examples as a useful starting point, not a universal hard minimum. Keep instructions consistent with the behavior in your examples.
- Costs: training, hosting, bad data (overfit, underfit, bias), retune when the base model or the data changes, and a model that gets worse outside the fine-tuned task.

```json
{"messages": [{"role": "system", "content": "You are a friendly travel advisor for Margie's Travel."}, {"role": "user", "content": "What's a good beach destination in Europe?"}, {"role": "assistant", "content": "For a beautiful European beach experience, consider the Algarve in southern Portugal."}]}
```

The screenshot retains an earlier `gpt-4.1` job. The [reviewed fine-tuning exercise](06-exercise-fine-tune.md) uses `gpt-5`, a small supplied JSONL fixture, and a Developer deployment for evaluation.
