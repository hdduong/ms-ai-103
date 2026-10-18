# 02 Understand Foundry agent publishing options

[Understand Foundry agent publishing options](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/2-understand-publishing-options?pivots=text)

- Publish turns the agent into an Agent Application: a stable URL, its own Entra identity, and user data that is not shared with other users. A new version keeps the same URL.
- Microsoft 365 publish creates a Bot Service, an Entra app, and a package, then lists the agent in the Teams store.
- **Shared** scope shows under Your agents immediately. **Organization** scope shows under Built by your org after an admin approves it.
- Other channels: a web preview, the REST endpoint, or Bot Service (Slack, Telegram, Twilio SMS, Facebook).
- The published identity does not inherit project permissions. Re-grant RBAC for Azure AI Search and the other tools.
- You need Azure AI Project Manager on the project, Azure AI User on the agent app, rights to create a bot and an Entra app, a tenant that allows custom apps, and an Agent 365 license for Defender discovery and threat detection.
