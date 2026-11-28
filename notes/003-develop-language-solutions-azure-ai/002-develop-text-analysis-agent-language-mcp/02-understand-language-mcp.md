# 02 Understand the Language MCP server

[Understand the Azure Language MCP server](https://learn.microsoft.com/en-us/training/modules/develop-text-analysis-agent-language-mcp/02-understand-language-mcp)

MCP is the same protocol as [path 002 module 003](../../002-develop-ai-agents-azure/003-connect-agent-to-mcp-tools/README.md): a host runs the agent, a client holds the connection, and a server publishes tools. The agent discovers the catalog at runtime.

The Azure Language MCP server exposes:

| Tool area | Job |
| --- | --- |
| Language detection | Which language the text is in |
| Named entity recognition | People, places, organizations, dates, quantities |
| PII redaction | Find and mask personal details |
| Text Analytics for health | Diagnoses, medications, symptoms in clinical text |

Sentiment and key phrases still appear for older apps. The lab's agent can still call `detect_sentiment_from_text`.

You do not write the if-statement that picks the tool. The model matches the prompt to the tool descriptions. One prompt can call language detection and entity recognition in the same turn.

```text
https://{foundry-resource-name}.cognitiveservices.azure.com/language/mcp?api-version=2025-11-15-preview
```

There is also a local MCP server in the [Azure Language samples](https://github.com/Azure-Samples/ai-language-samples). The lab uses the remote one from the Foundry catalog.
