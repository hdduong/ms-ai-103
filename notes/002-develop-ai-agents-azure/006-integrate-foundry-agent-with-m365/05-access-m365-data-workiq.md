# 05 Access Microsoft 365 data with Work IQ

[Access Microsoft 365 data with Work IQ](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/5-access-m365-data-workiq?pivots=text)

- Work IQ grounds an agent in workplace data: emails, meetings, SharePoint and OneDrive files, Teams messages, and people. The agent uses it as a tool, so you do not build search for each source.
- Foundry connects to the **remote Work IQ MCP** tool from the catalog: **Tools > Connect a tool > Catalog > Work IQ MCP**.
- Setup: an admin provisions the Work IQ service principal and a single-tenant app with the delegated **`WorkIQAgent.Ask`** permission plus admin consent and a client secret. Foundry's OAuth redirect URL goes back on the app as a **Web** redirect URI.
- It acts as the signed-in user (delegated). Admin consent does not widen that user's access. Application-only auth is not supported.
- Roles: **Foundry Project Manager** creates the connection. The developer, the runtime identity, and OAuth users need **Foundry User** on the project.
- Billing: **Copilot Credits** for Work IQ API usage. Connector-backed tools have their own licensing; the [full integration guide](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/work-iq#prerequisites) distinguishes these connection paths.
- Keep the client secret in a secret store and rotate it. Never in instructions or code.

| Decision | What it allows |
| --- | --- |
| Admin consent | The app may request `WorkIQAgent.Ask` in the tenant |
| User sign-in | The connection acts with that user's own access |
| Tool-call approval | This one proposed call may run |

## Lab: workplace-intelligence-agent

Apply these concepts in the optional [Work IQ exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05b-work-iq-integration.html), using its [remote-MCP instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/05b-work-iq-integration.md). The example combines one tenant-provisioning command with portal configuration and prompts. It uses a Foundry project with a deployed model, a Microsoft 365 account with known workplace data, and the roles and billing described above.

### Make the Work IQ API available to the client app

Check **Entra ID > Enterprise applications** for the public Work IQ application ID below. If it already exists, reuse it. Otherwise, an authorized **Global Administrator** performs this one-time tenant setup; replace `YOUR_TENANT_ID` with the Microsoft 365 tenant ID:

```azurecli
az login --tenant "YOUR_TENANT_ID"
az ad sp create --id fdcc1f02-fc51-4226-8753-f668596af7f7
```

`az login` selects the tenant where the account's data lives. `az ad sp create` provisions that tenant's instance of the **Work IQ service**, so Work IQ appears when adding API permissions. The returned `appId` should match the supplied ID; this is not the client ID of the app you create next. Resolve provisioning errors before proceeding ([tenant setup](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/enable-work-iq)).

Create or reuse a **single-tenant app registration** in that tenant. Add the delegated `WorkIQAgent.Ask` permission, grant admin consent, and create or reuse a valid client secret. Keep the secret **Value**, not its Secret ID, for the connection. Ordinary tool use does not require the administrator role.

### Configure the OAuth return path

In **Foundry > Tools > Connect a tool > Catalog > Work IQ MCP**, configure:

| Field | Value |
| --- | --- |
| Name | `work-iq-mcp` |
| Server URL | `https://workiq.svc.cloud.microsoft/mcp` |
| Client ID / secret | From the app registration |
| Authorization URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize` |
| Token and refresh URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token` |
| Scopes | `api://workiq.svc.cloud.microsoft/WorkIQAgent.Ask offline_access` |

The scope uses the application ID URI documented in the [Work IQ permissions reference](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/permissions) and [Foundry MCP quickstart](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/quickstart/foundry). The pinned exercise spells its resource prefix with the service's application ID instead. `offline_access` requests refresh capability so the delegated connection can renew access.

Replace `{tenant-id}` in all three URLs with the app's tenant ID. Save the connection, copy the generated OAuth redirect URL, and register that **exact** URL as a **Web** redirect URI on the app. Preserve existing redirect URIs when reusing an app. Without this return path, user authorization cannot complete even if the client ID and secret are valid.

### Give the agent a retrieval task

Create `workplace-intelligence-agent`, select the deployed model, add the configured Work IQ tool, and save these exercise instructions:

```text
You are a workplace intelligence assistant.
Use Work IQ to answer questions about the signed-in user's Microsoft 365 meetings, email, and Teams messages.
Keep responses concise and include source links when available.
If no matching data is found, say so. Don't invent workplace information.
If a tool call fails, explain the failure instead of presenting it as no data.
For this lab, only read and summarize information. Don't create, send, update, or delete Microsoft 365 content.
```

The instructions identify the source, require links, and distinguish missing records from a failed call. The read-and-summarize instruction describes this exercise's behavior; `WorkIQAgent.Ask` itself can cover read and write operations, so review the proposed tool and arguments before approval.

Ask the prompts below, sign in when requested, and inspect the proposed tool call before approving it:

```text
Use Work IQ to summarize my upcoming meetings for today.
Use Work IQ to find three recent emails I received this week. List the subject, sender, and date, and include source links when available.
Use Work IQ to find recent Teams messages about [a topic I recognize] from the past month. Summarize the discussion and include source links.
```

### Check the result and the user context

Follow the execution through the playground: your question becomes a Work IQ request, the delegated connection supplies the authorized user context, Work IQ returns accessible records, and the agent summarizes them. Compare at least one result with that user's calendar, mailbox, or Teams conversation, including its date and source link. Try a time range with known data; a no-data reply alone does not establish a working connection.

Repeat a known-content query with another authorized test user who lacks access to that content. The tool should honor that user's permissions. Admin consent, user sign-in, and call approval do not grant access to a document the user cannot otherwise read.

When finished, remove the lab-only agent, connection, and secret/app registration you no longer need. Preserve shared connections, existing app registrations and redirect URIs, and the tenant's Work IQ service principal.

Publishing to Teams in [unit 03](03-publish-agent-foundry-portal.md) lets people reach the agent. This lesson's Work IQ connection lets the agent retrieve workplace context on their behalf. The two features have separate setup and billing.
