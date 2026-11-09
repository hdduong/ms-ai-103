# 05 Access Microsoft 365 data with Work IQ

[Access Microsoft 365 data with Work IQ](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/5-access-m365-data-workiq?pivots=text)

- Work IQ grounds an agent in workplace data: emails, meetings, SharePoint and OneDrive files, Teams messages, and people. The agent uses it as a tool, so you do not build search for each source.
- Foundry connects to the **remote Work IQ MCP** tool from the catalog: **Tools > Connect a tool > Catalog > Work IQ MCP**.
- Setup: an admin provisions the Work IQ service principal and a single-tenant app with the delegated **`WorkIQAgent.Ask`** permission plus admin consent and a client secret. Foundry's OAuth redirect URL goes back on the app as a **Web** redirect URI.
- It acts as the signed-in user (delegated). Admin consent does not widen that user's access. Application-only auth is not supported.
- Roles: **Foundry Project Manager** creates the connection. The developer, the runtime identity, and OAuth users need **Foundry User** on the project.
- Billing: **Copilot Credits** for Work IQ API usage. Connector-backed tools have their own licensing.
- Keep the client secret in a secret store and rotate it. Never in instructions or code.

| Decision | What it allows |
| --- | --- |
| Admin consent | The app may request `WorkIQAgent.Ask` in the tenant |
| User sign-in | The connection acts with that user's own access |
| Tool-call approval | This one proposed call may run |

## Lab: workplace-intelligence-agent

Lab (optional): [Connect Work IQ workplace intelligence](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05b-work-iq-integration.html). Portal and OAuth, no Python.

1. One-time tenant setup, as an admin. Check **Entra ID > Enterprise applications** first and reuse it if it exists.

```azurecli
az login --tenant <your-microsoft-365-tenant-id>
az ad sp create --id fdcc1f02-fc51-4226-8753-f668596af7f7
```

2. App registration: single-tenant, delegated `WorkIQAgent.Ask`, admin consent, a client secret.
3. Foundry connection:

| Field | Value |
| --- | --- |
| Name | `work-iq-mcp` |
| Server URL | `https://workiq.svc.cloud.microsoft/mcp` |
| Client ID / secret | From the app registration |
| Authorization URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize` |
| Token and refresh URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token` |
| Scopes | `offline_access fdcc1f02-fc51-4226-8753-f668596af7f7/WorkIQAgent.Ask` |

4. Save, copy the generated redirect URL, and add it to the app as a Web redirect URI.
5. Create `workplace-intelligence-agent`, add the Work IQ tool, and set:

```text
You are a workplace intelligence assistant.
Use Work IQ to answer questions about the signed-in user's Microsoft 365 meetings, email, and Teams messages.
Keep responses concise and include source links when available.
If no matching data is found, say so. Don't invent workplace information.
If a tool call fails, explain the failure instead of presenting it as no data.
For this lab, only read and summarize information. Don't create, send, update, or delete Microsoft 365 content.
```

6. Sign in when asked, look at the proposed call, approve it, and check one answer against your real calendar or mail:

```text
Use Work IQ to summarize my upcoming meetings for today.
Use Work IQ to find three recent emails I received this week. List the subject, sender, and date, and include source links when available.
Use Work IQ to find recent Teams messages about [a topic I recognize] from the past month. Summarize the discussion and include source links.
```

- "No data" is not proof the connection works. A tool result you can check against the source is.
- Publishing to Teams (unit 03) lets people reach the agent. Work IQ lets the agent read workplace data. Separate setup, separate billing.
