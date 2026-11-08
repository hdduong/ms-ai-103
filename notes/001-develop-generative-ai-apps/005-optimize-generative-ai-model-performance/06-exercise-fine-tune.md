# 06 Exercise: Fine-tune a language model

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/optimize-generative-ai-model-performance/6-exercise)

> Reviewed **2026-11-07**. This official exercise uses the **Foundry portal and a JSONL training file**. It does not supply a Python fine-tuning program. Allow about **90 minutes**; training itself can take more than an hour.

## Setup and baseline

Create a project in **New Foundry** with an Azure subscription that can create the required resources. The pinned lab fine-tunes **gpt-5** and recommends **North Central US** or **Sweden Central**. Model, training, and deployment availability can change; check the model's current supported regions before creating resources.

Deploy the base model, open its playground, and compare generic assistant instructions with the following instructions used by the exercise:

```text
You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
You should not provide any hotel, flight, rental car or restaurant recommendations.
Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
```

Use the same travel questions before and after tuning: where to stay in Rome, affordable restaurants, local delicacies, weather, and getting around. The objective is to test whether the assistant consistently follows the intended style and recommendation restrictions, not simply whether it sounds fluent.

## Training data supplied by Microsoft

Download the [exact reviewed `travel-finetune-hotel.jsonl`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/data/travel-finetune-hotel.jsonl). This small lab fixture contains **10 examples**. It demonstrates the workflow; it is not evidence that ten examples are sufficient for a production use case.

Each physical line is one JSON object containing a conversation. This is the first record, shown in valid one-line JSONL form:

```jsonl
{"messages": [{"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."}, {"role": "user", "content": "What's a must-see in Paris?"}, {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}]}
```

The `system` message defines the desired behavior, the `user` message is an example input, and the `assistant` message is the behavior to learn. Review every record for consistency with the instructions before training. Keep independent evaluation examples separate from the training set.

## Portal settings

Open **Fine-tune**, create a job, and use the settings from the pinned exercise:

| Setting | Lab value |
| --- | --- |
| Base model | `gpt-5` |
| Fine-tuning method | Supervised |
| Training type | Standard |
| Training data | Upload `travel-finetune-hotel.jsonl` |
| Model suffix | `ft-travel` |
| Automatic deployment | Yes |
| Deployment type | Developer |
| Other job parameters | Defaults offered by the lab's portal flow |

Submit the job and inspect its status and logs. When training succeeds, open the fine-tuned model's playground. If automatic deployment failed after successful training, deploy that completed model manually rather than starting another training job.

Use the same instructions and test questions as the baseline. Check both compliance with the restrictions and usefulness of the remaining travel advice. Fine-tuning can improve consistency; it does not guarantee that every response obeys a rule.

## Developer deployment caveat

The current deployment reference describes **Developer** as an evaluation-only deployment with a **24-hour lifetime**, no availability SLA, and no data-residency guarantee. Do not treat the lab's automatically deployed model as a persistent production endpoint. See [Foundry deployment types](https://learn.microsoft.com/en-us/azure/ai-foundry/foundry-models/concepts/deployment-types).

## Lesson takeaways

- Start with a clear baseline and prompt before paying for fine-tuning.
- Fine-tuning adapts learned behavior and style; use retrieval when the problem is access to current or private information.
- Training and inference instructions should express the same intended behavior.
- This lab deliberately prohibits hotel/restaurant recommendations. The [Margie's Travel tool lab](../004-use-generative-ai-tools/07-exercise-use-tools.md) has a different scenario that answers questions about company offerings. Do not mix those instructions or datasets accidentally.
- Delete only the lab's disposable resources when no longer needed.

## Source and attribution

Adapted from Microsoft's [Fine-tune a language model](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/04b-finetune-model.md) and its [training data](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/data/travel-finetune-hotel.jsonl), commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md). JSONL structure and the source workflow were checked; no training job or deployment was run during this review.
