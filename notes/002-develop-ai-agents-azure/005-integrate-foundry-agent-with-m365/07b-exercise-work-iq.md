# Optional exercise — Connect Work IQ workplace intelligence

[Current exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05b-work-iq-integration.html) · [Pinned instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/05b-work-iq-integration.md)

Reviewed **2026-11-07**. The current optional lab uses the **remote Work IQ MCP server in Foundry**. It is a portal/OAuth exercise with one Azure CLI provisioning command, not a Python or Node.js application exercise.

## Prerequisites

Use a Microsoft 365 account with workplace data, a Foundry project/model, and **Copilot Credits usage-based billing enabled for Work IQ API access**. The lab requires **Foundry Project Manager** to create the connection and **Foundry User** for the developer, runtime, and OAuth users on the project. Tenant setup and consent require the administrator role described by the exercise; ordinary use doesn't require that administrator role.

The current authoritative setup references are [Enable your tenant for Work IQ](https://learn.microsoft.com/microsoft-365/copilot/extensibility/work-iq/enable-work-iq) and [Connect a Foundry project to Work IQ](https://learn.microsoft.com/microsoft-365/copilot/extensibility/work-iq/mcp/quickstart/foundry).

## Exercise command: provision the service principal

Check **Entra ID > Enterprise applications** for the Work IQ application ID below first. This is a **one-time tenant setup**. If it exists, reuse it; don't provision a duplicate.

With an authorized administrator signed in to the intended Microsoft 365 tenant:

```azurecli
az login --tenant <your-microsoft-365-tenant-id>
az ad sp create --id fdcc1f02-fc51-4226-8753-f668596af7f7
```

The returned `appId` should match the supplied ID. Resolve any error other than an already-existing principal before continuing. The ID is the public Work IQ application ID from the exercise, not your new client application's ID.

## App registration and OAuth configuration

Create or reuse an approved **single-tenant** app registration. Add the delegated **WorkIQAgent.Ask** permission for the Work IQ API and grant admin consent. Create a client secret only if an approved unexpired one isn't already available; keep its **Value** in an approved secret store.

In **Foundry > Tools > Connect a tool > Catalog > Work IQ MCP**, use:

| Field | Value |
| --- | --- |
| Name | `work-iq-mcp` |
| Server URL | `https://workiq.svc.cloud.microsoft/mcp` |
| Client ID | Your app registration's application/client ID |
| Client secret | Your app registration's secret value |
| Authorization URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize` |
| Token URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token` |
| Refresh URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token` |
| Scopes | `offline_access fdcc1f02-fc51-4226-8753-f668596af7f7/WorkIQAgent.Ask` |

Replace `{tenant-id}` with the same tenant ID used for the app. Save, copy Foundry's generated OAuth redirect URL, and add it to the app registration as an exact **Web redirect URI**. Preserve existing redirect URIs when reusing an app. Admin consent, user authorization, and tool-call approval are separate steps.

## Agent instructions from the exercise

Create `workplace-intelligence-agent`, add the configured Work IQ tool, and save:

```text
You are a workplace intelligence assistant.
Use Work IQ to answer questions about the signed-in user's Microsoft 365
meetings, email, and Teams messages.
Keep responses concise and include source links when available.
If no matching data is found, say so. Don't invent workplace information.
If a tool call fails, explain the failure instead of presenting it as no data.
For this lab, only read and summarize information. Don't create, send,
update, or delete Microsoft 365 content.
```

## Run and inspect

Ask a question in the playground, complete user authorization if requested, and inspect the proposed tool call before approving it:

```text
Use Work IQ to summarize my upcoming meetings for today.

Use Work IQ to find three recent emails I received this week.
List the subject, sender, and date, and include source links when available.

Use Work IQ to find recent Teams messages about [a topic I recognize]
from the past month. Summarize the discussion and include source links.

Using Work IQ, find any action items assigned to me in that discussion.
Include deadlines only if they are explicitly stated in the source.
```

Use a time period with known data. Connectivity is demonstrated by a successful tool result and at least one answer checked against the actual calendar, email, or Teams source. A no-data reply or a model-generated summary by itself doesn't establish connectivity. This delegated flow respects the user's existing access; it doesn't grant organization-wide access.

## Cleanup and exam distinction

Remove the lab agent, its lab-only Work IQ connection, and the lab-only client secret/app registration when no longer needed. Preserve shared connections, app registrations, redirect URIs, and the tenant Work IQ service principal. Review a dedicated Azure resource group before deleting it. Model and Work IQ calls may incur usage charges.

**Teams/Copilot publishing** lets people reach your agent. **Work IQ** lets your agent retrieve permitted workplace context. They can be used together but are separate integrations with separate setup and billing requirements.

Command, prompts, and configuration adapted from MicrosoftLearning/mslearn-ai-agents at `4d6a32629722927bd5f1034e0b5396e0da4e962c` (2026-11-04). See [third-party notices](../../../THIRD_PARTY_NOTICES.md).
