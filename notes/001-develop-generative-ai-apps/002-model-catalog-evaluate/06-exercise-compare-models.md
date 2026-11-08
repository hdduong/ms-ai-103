# 06 Exercise: Explore and compare models

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/model-catalog-evaluate/6-exercise)

> Reviewed **2026-11-07**. This official exercise is **portal-only**: it supplies prompts and an evaluation workflow, not Python application code. Allow about **45 minutes**.

## Setup

Use an Azure subscription with permission to create Foundry resources. Create a project in the **New Foundry** experience and select a region that supports the required models. The pinned lab compares **gpt-5.2** and **gpt-5-mini**; deploy both and record their deployment names.

## Compare before building

1. Browse model cards in **Discover → Models**. Check supported tasks, context size, endpoints, features, and deployment availability.
2. Inspect **Benchmarks** and the leaderboard. Compare quality, safety, cost, and throughput for your intended task; one strong score does not settle all four tradeoffs.
3. Open the playground's model comparison, select both deployments, and give them the same fox/chicken/grain river-crossing puzzle from the official lab.
4. Ask `Explain your reasoning.` Compare the observable answer, correctness, and explanation. This is a request for a useful explanation, not access to a model's private internal reasoning.

Keep the prompt and instructions consistent when comparing models. Change one experimental variable at a time so a difference in results has an interpretable cause.

## Run the travel-assistant evaluation

| Setting | Value used by the official lab |
| --- | --- |
| Evaluation target | Model: the `gpt-5.2` deployment |
| Data | Generate synthetic data |
| Number of rows | `45` |
| Generation model | `gpt-5.2` |
| Generation prompt | `Create various travel related questions, and include some content safety and security tests` |
| Seed data | Leave blank |
| Developer prompt | `You are a helpful travel assistant that provides accurate, detailed, and practical travel advice to help users plan their trips.` |
| Criteria | Remove **Agent** and **Safety** criteria, retaining the remaining criteria as the lab directs |
| Evaluation name | `travel-assistant-eval` |

Submit the run, inspect aggregate and individual results, and use **Analyze results** to find patterns among failures. Read the actual question, response, evaluator explanation, and score together.

**Interpretation:** The generated dataset includes safety/security questions, but this lab explicitly removes the **Safety evaluators**. Therefore this run does **not** establish a measured safety pass rate. A refusal can be the desired response to an unsafe question even if a general quality evaluator marks it poorly. Use suitable safety evaluators in a separate, deliberately configured evaluation.

## Lesson takeaways

- Benchmarks are useful for shortlisting; evaluation on your own representative cases determines suitability for your application.
- Synthetic cases are a starting point. Review their relevance and add real, approved test cases and edge cases.
- Model quality, cost, latency/throughput, and safety require separate interpretation.
- Evaluation names, input data, prompts, deployments, and evaluator settings are part of a reproducible comparison.

Delete the lab's disposable resource group when finished if you no longer need its deployments.

## Source and attribution

Adapted from Microsoft's [Explore and compare models](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/02-model-catalog-evaluation.md), commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md). The workflow was checked against source; an Azure evaluation was not run during this review.
