# 05 Exercise: integrate Web IQ with an agent

Reviewed **2026-11-07**.

- [Official exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04b-web-iq.html)
- [Pinned exercise source](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/04b-web-iq.md)
- [Source attribution and license](../../../THIRD_PARTY_NOTICES.md)

**Exercise type:** portal configuration and prompts. This exercise contains no Python application or package installation; do not substitute invented SDK code for its steps.

## Run the exercise

1. Create Web IQ using an eligible paid Azure subscription. Open its portal and test a query.
2. Create/select a Foundry project and a model that supports tools.
3. Create a music assistant. Remove its existing Web search tool to isolate this comparison.
4. Ask about a band, then about its upcoming concerts. Save the baseline answer.
5. Add Web IQ from the catalog using the profile's API key. Review the exercise's automatic approval setting and save the agent.
6. Repeat the questions and request an image search. Inspect **Traces** for actual Web IQ calls and compare the supporting sources.

## What to learn

The tool provides external evidence at request time; it does not retrain the model. Evaluate the retrieved dates and sources, then whether the answer supports its claims with that evidence.

Remove only resources created for this exercise when finished. Shared projects and resources may support other labs.

[Back to module](README.md)
