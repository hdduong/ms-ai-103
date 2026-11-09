# 04 Fine-tune a model for consistent behavior

[Fine-tune a model for consistent behavior](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/4-fine-tune-model?pivots=text)

- Fine-tuning changes weights so style, tone, or format stay consistent when prompts and few-shot examples do not.
- Foundry uses **LoRA**: it updates a small set of parameters, not the whole model.
- Measure a baseline first. Without one you cannot tell if the fine-tune helped.
- Use it for brand voice, a schema the model keeps missing, shorter prompts, distillation onto a smaller model, or better tool selection.
- Methods: **SFT** (prompt and response pairs), **RFT** (a grader rewards better answers), **DPO** (preferred versus rejected pairs). SFT then DPO is a valid sequence. The portal labels SFT as **Supervised**.

![Build > Fine-tune. A completed Supervised job on gpt-4.1.](images/04-finetune.png)
- Training file is **JSONL**. Each line is `messages` with system, user, and assistant. Keep the same system message in training and at inference. Hundreds of examples is the starting point. A blank system message hurts accuracy.
- Costs: training, hosting, bad data (overfit, underfit, bias), retune when the base model or the data changes, and a model that gets worse outside the fine-tuned task.

```json
{"messages": [{"role": "system", "content": "You are a friendly travel advisor for Margie's Travel."}, {"role": "user", "content": "What's a good beach destination in Europe?"}, {"role": "assistant", "content": "For a beautiful European beach experience, consider the Algarve in southern Portugal."}]}
```

## Lab: fine-tune gpt-5 in the portal

Lab: [Fine-tune a language model](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04b-finetune-model.html). Portal only. Use North Central US or Sweden Central. Training can take over an hour.

1. **Baseline.** Deploy `gpt-5`. In the playground, use these instructions and ask about Rome: where to stay, cheap restaurants, local food, weather, getting around.

```text
You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
You should not provide any hotel, flight, rental car or restaurant recommendations.
Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
```

2. **Training data.** [`travel-finetune-hotel.jsonl`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/data/travel-finetune-hotel.jsonl) has 10 examples. The first line:

```json
{"messages": [{"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."}, {"role": "user", "content": "What's a must-see in Paris?"}, {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}]}
```

3. **Job.** **Fine-tune** > create a job:

| Setting | Lab value |
| --- | --- |
| Base model | `gpt-5` |
| Method | Supervised |
| Training type | Standard |
| Training data | Upload `travel-finetune-hotel.jsonl` |
| Suffix | `ft-travel` |
| Automatic deployment | Yes, **Developer** type |

4. **Compare.** Ask the fine-tuned model the same questions with the same instructions. Check that it skips hotel and restaurant picks and keeps asking questions.

The **Developer** deployment is for evaluation only. It is deleted after 24 hours and has no SLA.
