# 03 Explore Foundry IQ

[Explore Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/3-foundry-iq?pivots=text)

- Foundry IQ is a managed knowledge layer on Azure AI Search. Many agents share one knowledge base. You do not stand up a vector database per agent.
- A knowledge base is a business domain, such as "Product Documentation", not a container name. SharePoint, Blob, and OneLake can sit in the same base.
- Indexed sources follow ingestion, processing, indexing, and refresh. Remote sources query the source at request time. The source type determines which path applies.
- The service retrieves and ranks grounding information and returns source references. Query planning and answer synthesis depend on the selected retrieval configuration and API version.
- The agent connects over MCP. The complete knowledge-base URL includes an API version, and the connection must authenticate to Azure AI Search.

![Knowledge (Foundry IQ). knowledgebase301 on the project's Azure AI Search connection, with one active knowledge source.](images/03-knowledge-bases.png)

## Knowledge base as an MCP tool

In the [Foundry IQ exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04-integrate-agent-with-foundry-iq.html), the portal creates the tool binding when you attach a knowledge base. Its project connection stores Search authentication. [Unit 04](04-data-requirements.md#lab-blob-source-for-product-expert-agent) configures that connection, and [unit 05](05-configure-retrieval.md#lab-ask-then-approve-the-knowledge-base-call) calls the saved agent.

The Learn unit also demonstrates creating the tool in code. This definition fragment makes the two connection inputs explicit; it assumes `project_client` is initialized and the knowledge base and authenticated project connection already exist:

```python
import os
from azure.ai.projects.models import MCPTool, PromptAgentDefinition

knowledge_tool = MCPTool(
    server_label="product-docs",
    server_url=os.environ["KNOWLEDGE_BASE_MCP_URL"],
    project_connection_id=os.environ["SEARCH_PROJECT_CONNECTION_ID"],
    require_approval="always",
)
agent = project_client.agents.create_version(
    agent_name="product-support-agent",
    definition=PromptAgentDefinition(
        model=os.environ["MODEL_DEPLOYMENT_NAME"],  # deployment name
        instructions="Search the product knowledge base and cite the sources. Say when evidence is missing.",
        tools=[knowledge_tool],
    ),
)
```

| Value | Where it comes from | Why it matters |
| --- | --- | --- |
| `KNOWLEDGE_BASE_MCP_URL` | The configured Search knowledge base; include `/mcp?api-version=<supported-version>` | Selects the knowledge base and its supported retrieval API |
| `SEARCH_PROJECT_CONNECTION_ID` | The existing project connection configured for Search | Supplies the separate Search authentication |
| `require_approval="always"` | Your agent configuration | Causes the client to receive approval requests before retrieval |

The bare URL in the training snippet leaves out configuration required by the [Search retrieval API](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-retrieve). Foundry credentials do not automatically grant access to Search. Use the portal-based route in the next lessons to follow the exercise without duplicating this agent definition.
