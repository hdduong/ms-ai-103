# 03 Connect and use the Language MCP server

[Connect and use the Language MCP server](https://learn.microsoft.com/en-us/training/modules/develop-text-analysis-agent-language-mcp/03-connect-use-language-mcp)

In the portal: add **Azure Language** from the tool catalog, then attach it to the agent. In code the same connection is one `MCPTool`.

```python
from azure.ai.projects.models import MCPTool

mcp_tool = MCPTool(
    server_label="azure-language",
    server_url="https://{foundry-resource-name}.cognitiveservices.azure.com/language/mcp?api-version=2025-11-15-preview",
    require_approval="always",  # pause before each Language call
)
```

## Lab: ask Text-Analysis-Agent

Lab: [Build a text analysis agent](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/02-language-agent.html). Create `Text-Analysis-Agent`, add the Language tool, and save. `FOUNDRY_ENDPOINT` here is the **project** endpoint, because `AIProjectClient` wants it.

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()
project_client = AIProjectClient(
    endpoint=os.environ["FOUNDRY_ENDPOINT"],  # project endpoint, with /api/projects
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()
agent_name = os.environ["AGENT_NAME"]  # Text-Analysis-Agent

prompt = input("User prompt: ")
# Run the saved agent; it picks the Language tools
response = openai_client.responses.create(
    input=[{"role": "user", "content": prompt}],
    extra_body={"agent_reference": {"name": agent_name, "type": "agent_reference"}},
)
print(response.output_text)
# The JSON lists the tools the agent was offered and the ones it called
print(response.model_dump_json(indent=2))
```

Try `Extract named entities from the following text: "Pierre and I went to Paris on July 14th."` The tool name in the JSON is `extract_named_entities_from_text`. A review that also asks if the text is positive or negative calls `detect_sentiment_from_text` as well.
