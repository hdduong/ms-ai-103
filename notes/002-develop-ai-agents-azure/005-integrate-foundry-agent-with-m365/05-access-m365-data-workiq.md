# 05 Access Microsoft 365 data with Work IQ

[Access Microsoft 365 data with Work IQ](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/5-access-m365-data-workiq?pivots=text)

Reviewed **2026-11-07** against the current unit and [optional exercise](07b-exercise-work-iq.md).

- Work IQ is Microsoft's workplace intelligence layer. It can ground agents in emails, meetings, SharePoint/OneDrive files, Teams messages, and people, with permission-aware context.
- The current lesson connects **remote Work IQ MCP** from the Foundry catalog. Node.js and a local `@microsoft/workiq` process aren't prerequisites for this lab.
- Requests use **delegated authentication** as the user who authorizes the connection. Admin consent approves the app's permission; it doesn't expand that user's Microsoft 365 access. Application-only authentication isn't supported for this connection.
- Work IQ API usage requires **usage-based billing with Copilot Credits**. Check separate licensing requirements for connector-backed tools. Don't substitute the Teams/Copilot publishing lab's license requirements for these API requirements.
- Roles in this unit: **Foundry Project Manager** creates the connection; the developer, runtime identity, and OAuth users need **Foundry User** on the project.

## Configure the connection

1. Reuse an approved Work IQ connection if one exists. Otherwise, an administrator provisions the Work IQ service principal and grants a single-tenant app the delegated `WorkIQAgent.Ask` permission with admin consent.
2. In Foundry, choose **Tools > Connect a tool > Catalog > Work IQ MCP**. Configure the app client ID, client secret, and tenant OAuth authorization/token/refresh URLs.
3. Copy the OAuth redirect URL produced by Foundry and register that **exact URL** as a Web redirect URI on the Entra app.
4. Add the configured Work IQ tool to the agent and test with your signed-in account. Review tool-call arguments and approve the relevant read request.

```text
Remote MCP endpoint: https://workiq.svc.cloud.microsoft/mcp
Delegated permission: WorkIQAgent.Ask
Scopes: offline_access fdcc1f02-fc51-4226-8753-f668596af7f7/WorkIQAgent.Ask
```

Keep secrets in an approved secret store, rotate them, and don't put them in agent instructions or source control. The current unit says connection values can't be edited after creation; correct mistakes by replacing only a lab-owned connection, preserving shared connections.

## Know the three separate decisions

| Decision | What it allows |
| --- | --- |
| Administrator consent | The app may request the delegated Work IQ permission in the tenant |
| User authorization | The connection acts with that signed-in user's permitted data access |
| Tool-call approval | The agent may execute the particular proposed request |

A failed tool call isn't proof that no matching data exists. Check the tool result, the account and permissions, the date range, and source links. See the [complete optional exercise with the tenant setup command and OAuth values](07b-exercise-work-iq.md).
