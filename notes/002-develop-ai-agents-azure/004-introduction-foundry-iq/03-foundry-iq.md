# 03 Explore Foundry IQ

[Explore Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/3-foundry-iq?pivots=text)

- Foundry IQ is a knowledge platform on Azure AI Search. Build the knowledge base once. Many agents connect to it.
- A knowledge base is a business domain (product docs, HR policy), not a storage account. SharePoint, blobs, OneLake, and an existing index can sit in one base.
- Adding a source: discover files, chunk and embed, index, reindex when files change.
- Retrieval picks keyword or semantic search, ranks results, and returns citations. You do not write that logic.
- The agent connects with an `MCPTool` whose URL is `{search_endpoint}/knowledgebases/{name}/mcp`.
- Share a base across agents. A support agent and a developer agent can both use product docs. An HR agent uses only the HR base.

```python
knowledge_tool = MCPTool(
    server_label="product-docs",
    server_url=f"{search_endpoint}/knowledgebases/product-documentation/mcp",
)
agent = project_client.agents.create_version(
    agent_name="product-support-agent",
    definition=PromptAgentDefinition(
        model="gpt-4.1",  # deployment name
        instructions="Answer product questions from the knowledge base. Cite the source.",
        tools=[knowledge_tool],
    ),
)
print(agent.name)
```
