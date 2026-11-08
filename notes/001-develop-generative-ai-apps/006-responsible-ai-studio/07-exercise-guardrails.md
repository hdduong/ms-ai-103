# 07 Exercise: Apply guardrails to prevent harmful output

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/responsible-ai-studio/7-exercise-content-filters)

> Reviewed **2026-11-07**. This is a **portal-only** exercise. The official lab does not supply SDK code. Allow about **25 minutes**.

## Setup and baseline

Create a project in the **New Foundry** experience with an Azure subscription and deploy **gpt-5.2** using the defaults available for the selected region.

Open the model playground and run the official lab's three test prompts: a request to help with a bank robbery, an offensive joke request, and an ambiguous question about a cut. Record whether each produces a refusal, a content-filter block, or an allowed response. See the [pinned exercise](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/06-Explore-content-filters.md) for the exact strings so the before/after test remains consistent.

The injury question illustrates ambiguity: a benign first-aid question may be misinterpreted as self-harm. Do not equate every flagged term with harmful intent.

## Create and attach the guardrail

1. Open **Build → Guardrails → Create** and the **Risk** section.
2. Configure the four categories **Hate**, **Violence**, **Sexual**, and **Self-harm** with **Highest blocking** as the exercise directs.
3. Add the controls, replacing existing controls when prompted in that workflow.
4. Continue to **Models**, select the `gpt-5.2` deployment used for the baseline, review the settings, and submit.
5. Open the guardrail details and verify that it is attached to the intended deployment.
6. Return to that deployment's playground and repeat the same prompts.

**Highest blocking** means the more restrictive setting; do not read it as “block only the highest-severity content.” The labels and available controls can vary as the portal changes.

## Interpret the results

| Observation | What it demonstrates |
| --- | --- |
| Assistant declines in ordinary response text | The model's own safety behavior can refuse a request |
| Service reports a content-filter block | A configured filtering control stopped input/output |
| Same refusal before and after configuration | The model may already have refused; this does not by itself show that the new guardrail failed |
| Benign question is blocked | A possible false positive to investigate against the application's requirements |
| One unsafe prompt is blocked | Evidence for that test case, not proof of coverage against every attack |

Responses can differ between runs. Compare the actual filter result and response, retain representative test cases, and assess useful answers as well as unwanted output. Content filtering supports the broader responsible AI process; it does not replace risk identification, evaluation, monitoring, or user feedback.

## Lesson takeaways

- A guardrail must be associated with the deployment that the application actually calls.
- Keep the model, prompt, and instructions fixed for a meaningful before/after comparison.
- Distinguish **model refusal**, **filter enforcement**, and **false positives**.
- Follow [Identify harms](03-identify-harms.md), [Measure harms](04-measure-harms.md), and [Operate responsibly](06-operate-responsibly.md) for the rest of the lifecycle.

When finished, remove only the disposable resource group created for this lab if it is no longer needed.

## Source and attribution

Adapted from Microsoft's [Apply guardrails to prevent output harmful content](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/06-Explore-content-filters.md), commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md). Portal instructions were checked against source; no live guardrail was created or tested during this review.
