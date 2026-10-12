# ms-ai-103

Study notes for the Microsoft Learn path [Develop generative AI apps on Microsoft Foundry](https://learn.microsoft.com/en-us/training/paths/develop-generative-ai-apps/) (titled *Develop generative AI apps in Azure* on the path page).

The path is intermediate. It is aimed at data scientists and AI engineers who already know basic AI concepts, Azure services, and a programming language. It covers building generative AI apps on [Microsoft Foundry](https://ai.azure.com) that use language models to talk with users.

## Modules

1. [Plan and prepare to develop AI solutions on Azure](https://learn.microsoft.com/en-us/training/modules/prepare-azure-ai-development/)
2. Select, deploy, and evaluate Microsoft Foundry models
3. [Develop a generative AI chat app with Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/foundry-sdk/)
4. Develop generative AI apps that use tools
5. Optimize generative AI model performance with Microsoft Foundry
6. Implement a responsible generative AI solution in Microsoft Foundry

## Sample unit: Choose an endpoint and SDK

Module 3, unit: [Choose an endpoint and SDK](https://learn.microsoft.com/en-us/training/modules/foundry-sdk/03-microsoft-foundry-sdk?pivots=text).

Before writing a chat app, pick an endpoint, a client SDK, and an authentication method. Each Foundry project exposes both a project endpoint and an Azure OpenAI endpoint.

| Choice | Endpoint shape | Client | Use it when |
| --- | --- | --- | --- |
| Foundry SDK | `https://{resource-name}.services.ai.azure.com/api/projects/{project-name}` | `AIProjectClient` from `azure-ai-projects`, then `get_openai_client()` | The app needs agents, evaluations, tracing, project connections, or models from the Foundry catalog |
| OpenAI SDK | `https://{resource-name}.openai.azure.com/openai/v1` | `OpenAI` (or `AzureOpenAI` when a specific Azure OpenAI API version is required) | Existing OpenAI code should keep working, or the workload is model inference only |

Production apps should authenticate with Microsoft Entra ID (`DefaultAzureCredential`, after `az login` for local runs). API keys and `OPENAI_BASE_URL` / `OPENAI_API_KEY` also work; keep keys in Azure Key Vault and out of source.

For chat, prefer the Responses API on new work. Chat Completions remains the compatible option across models and platforms.

Python packages for the Foundry path:

```bash
pip install azure-ai-projects azure-identity openai
```

The OpenAI-only path needs `openai` and `azure-identity` when using Entra ID.
