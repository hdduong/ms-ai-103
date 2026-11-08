# 07 Exercise: Prepare for an AI development project

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/prepare-azure-ai-development/7-exercise-explore-ai-foundry)

> Reviewed against Microsoft Learn and the official lab on **2026-11-07**. This is a **portal and VS Code exercise**; the official lab does not supply a Python application to implement.

## Goal and prerequisites

Create a Foundry project, deploy a model, test it, and connect the project to VS Code. Allow about **30 minutes**. The official lab requires an Azure subscription, VS Code, Git, Azure CLI, and Python **3.13.x**; its tested version is **3.13.12**. These are the lab's tested prerequisites, not a guarantee that every later Python version works with every SDK.

## Walkthrough

1. Open [Microsoft Foundry](https://ai.azure.com/), enable the **New Foundry** experience, and create a project. Choose a subscription, resource group, resource name, and a region that supports the lab's model and Responses API.
2. In **Discover → Models**, deploy **gpt-5.2** with the default settings offered for your region. Record the actual deployment name; SDK calls use that name.
3. In the playground, set these instructions:

   ```text
   You are an AI assistant that can provide information and advice about AI software development.
   ```

4. Test with:

   ```text
   Describe three key considerations for working with Large Language Models for AI application development.
   ```

5. Open **Manage** and inspect the relationship between the parent Foundry resource, its projects, deployed models, and shared tools/connections. Return to the project home page and identify its endpoints.
6. Install **Foundry Toolkit for VS Code**, sign in to Azure, select the project, and set it as the default project. Open **Developer Tools → Build → Model playground**, select the deployment, and repeat the prompt.

## Record the right connection details

| Detail | Purpose |
| --- | --- |
| Project endpoint | Foundry project APIs and supported model APIs through the project |
| Azure OpenAI endpoint | Direct OpenAI-compatible model calls; the later chat lab uses the `/openai/v1/` base URL |
| Model deployment name | Value supplied as `model=`; it can differ from the catalog model name |
| Authentication | The lab introduces keys and Microsoft Entra ID; the coding exercises use Entra ID through `DefaultAzureCredential` |

An Azure sign-in and an endpoint identify the caller and destination; the caller also needs permission to use the resource. Never copy a real key into notes or source control.

## Lesson takeaways

- A **resource** can contain multiple **projects**. Selecting a project establishes the context for development.
- Model availability and deployment options depend on the model, region, and subscription. Follow the current portal if a screenshot differs.
- The portal and VS Code playgrounds help test instructions before writing an application.
- The Python implementation starts in [Exercise: Create a chat app](../003-foundry-sdk/06-exercise-chat-app.md).

After the lab, remove only the disposable resource group you created for it when you no longer need it.

## Source and attribution

Adapted from Microsoft's [Prepare for an AI development project](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/01-Explore-ai-studio.md), pinned to commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md). Portal steps were reviewed; no Azure resources were provisioned during this review.
