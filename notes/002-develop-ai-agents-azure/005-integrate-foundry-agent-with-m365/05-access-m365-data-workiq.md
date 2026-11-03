# 05 Access Microsoft 365 data with Work IQ

[Access Microsoft 365 data with Work IQ](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/5-access-m365-data-workiq?pivots=text)

- Work IQ is a CLI and an MCP server over Microsoft 365 Copilot data: mail, meetings, SharePoint and OneDrive files, Teams messages, and people.
- It uses Graph as the signed-in user. You only see what that user can already open. It does not store the data. An admin can see the queries.
- Needs Node.js for the local CLI, a Microsoft 365 Copilot license, and admin consent for the Work IQ app.
- CLI for a one-off question or a script. MCP mode so Copilot in VS Code calls the same tools. Work IQ APIs are generally available as of June 2026, and a remote MCP endpoint is supported.

## Install and accept the license

```bash
npm install -g @microsoft/workiq
npx -y @microsoft/workiq mcp
workiq accept-eula
```

## Ask from the terminal

```bash
workiq ask -q "What requirements did Sarah share about the authentication feature?"
```

## MCP server in VS Code

```json
{
  "workiq": {
    "command": "npx",
    "args": ["-y", "@microsoft/workiq", "mcp"],
    "tools": ["*"]
  }
}
```
