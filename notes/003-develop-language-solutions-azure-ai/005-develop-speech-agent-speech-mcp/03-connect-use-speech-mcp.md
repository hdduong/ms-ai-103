# 03 Connect and use the Speech MCP server

[Connect and use the Speech MCP server](https://learn.microsoft.com/en-us/training/modules/develop-speech-agent-speech-mcp/03-connect-use-speech-mcp)

Portal connection fields include the resource key and **X-Blob-Container-Url**, the SAS URL from [02](02-understand-speech-mcp.md).

```python
from azure.ai.projects.models import MCPTool

mcp_tool = MCPTool(
    server_label="azure-speech",
    server_url="https://{foundry-resource-name}.cognitiveservices.azure.com/speech/mcp",
    require_approval="always",
)
```

## Lab: ask the speech agent

Lab: [Build a speech agent](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/05-azure-speech-mcp.html). Same client shape as the [Language agent](../002-develop-text-analysis-agent-language-mcp/03-connect-use-language-mcp.md#lab-ask-text-analysis-agent). `FOUNDRY_ENDPOINT` is the project endpoint. `AGENT_NAME` is the agent you saved.

```python
response = openai_client.responses.create(
    input=[{"role": "user", "content": prompt}],
    extra_body={"agent_reference": {"name": agent_name, "type": "agent_reference"}},
)
print(response.output_text)
```

"Transcribe this audio file" should call recognize. "Generate speech from this text" should call synthesize and return a blob link.
