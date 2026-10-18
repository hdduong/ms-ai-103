# 05 Access Microsoft 365 data with Work IQ

[Access Microsoft 365 data with Work IQ](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/5-access-m365-data-workiq?pivots=text)

- Work IQ is an MCP server over Microsoft 365 Copilot data: mail, meetings, SharePoint and OneDrive files, Teams messages, and people.
- CLI: `workiq ask -q "..."`. MCP mode: an assistant calls the same tools. Same data and the same permissions either way.
- Install with `npm install -g @microsoft/workiq` or `npx -y @microsoft/workiq mcp`. Run `workiq accept-eula` once.
- Needs Node.js, a Microsoft 365 Copilot license, and admin consent. It only sees what that user can already open. It does not store the data. Graph calls are auditable.
- Remote MCP is supported so a Foundry agent can call Work IQ without the CLI.
