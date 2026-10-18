# 07 Extend agent capabilities with tools

[Extend agent capabilities with tools](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/7-extend-agent-capabilities?pivots=text)

- The service runs the tool loop: pick a tool, call it, fold the result into the reply. You do not write that loop.
- Built-ins: code interpreter (sandbox Python), file search (vector store; PDF, docx, txt, md), Bing grounding (cites sources), Azure AI Search (an existing index, not an upload), OpenAPI 3.0.
- Also in the catalog: browser automation, computer use, image generation, SharePoint, Fabric, deep research, agent-to-agent, memory.
- MCP servers: remote (production), local (dev), or your own. A **toolbox** is one managed MCP endpoint. Update the toolbox and every agent sees it. No redeploy.
- Say in the instructions when to use each tool. Extra tools add latency. Start with built-ins.

```yaml
tools:
  - type: code_interpreter
  - type: bing_grounding
    bing_grounding:
      connection_id: your-connection-id
  - type: file_search
    file_search:
      vector_store_ids:
        - vectorstore-123
```
