# 03 Explore Foundry IQ

[Explore Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/3-foundry-iq?pivots=text)

- Foundry IQ is a managed knowledge layer on Azure AI Search. Many agents share one knowledge base. You do not stand up a vector database per agent.
- A knowledge base is a business domain, such as "Product Documentation", not a container name. SharePoint, Blob, and OneLake can sit in the same base.
- Indexed sources use ingestion, chunking/enrichment, embeddings where configured, and indexing. Remote sources use their own live retrieval path; not every knowledge source follows the same pipeline.
- Retrieval behavior depends on the knowledge source, service/API version, and configured reasoning effort. The service returns retrieved grounding information and references; LLM-based query planning and answer synthesis require the corresponding configuration.
- A knowledge base exposes MCP at `{search_endpoint}/knowledgebases/{name}/mcp?api-version=<supported-version>`. The URL needs the correct API version and Search authentication; it is not anonymously callable just because the agent has Foundry credentials.

![Knowledge (Foundry IQ). knowledgebase301 on the project's Azure AI Search connection, with one active knowledge source.](images/03-knowledge-bases.png)

## Knowledge base as an MCP tool

For the exercise, connect the knowledge base in the portal and call that saved agent. Its project connection stores Search authentication. [The complete exercise](06-exercise-foundry-iq.md) uses this route.

A definition-only example for an existing authenticated project connection is:

```python
import os
from azure.ai.projects.models import MCPTool, PromptAgentDefinition

knowledge_tool = MCPTool(
    server_label="product-docs",
    # Copy the complete knowledge-base MCP URL, including its supported api-version.
    server_url=os.environ["KNOWLEDGE_BASE_MCP_URL"],
    project_connection_id=os.environ["SEARCH_PROJECT_CONNECTION_ID"],
    require_approval="always",
)
agent = project_client.agents.create_version(
    agent_name="product-support-agent",
    definition=PromptAgentDefinition(
        model=os.environ["MODEL_DEPLOYMENT_NAME"],
        instructions="Answer product questions from the knowledge base. Cite the sources.",
        tools=[knowledge_tool],
    ),
)
```

This fragment assumes an initialized `project_client`, a provisioned knowledge base, and a compatible Search connection. The client must handle pending MCP approvals. Do not copy a bare URL or fabricate a connection ID.

The [current retrieval API documentation](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-retrieve) distinguishes the generally available extractive API from preview features such as LLM query planning/answer synthesis. Use the API version that supports your configured knowledge base. It also documents MCP authentication: a Microsoft Entra bearer token with appropriate Search data access, or a Search admin key. A Foundry project token and a Search data-plane token serve different resources.
