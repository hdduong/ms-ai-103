# 04 Build your first agent in Microsoft Foundry

[Build your first agent in Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/4-build-agent-azure-portal?pivots=text)

- Portal path: `https://ai.azure.com`, open the project, **Build > Agents**, **Create**.
- You set the name, a description, and a **deployed** model. The dropdown only lists deployments in this project.
- Instructions are the system prompt. Temperature and top p are on the same screen. Unit 06 is the exam detail for those numbers.
- **Playground** keeps the session history, so you can test more than one turn before you deploy.
- **Tools** (also **Build > Tools**) has three groups: **Configured** (Code Interpreter, File Search), **Catalog** (Bing, Azure AI Search, SharePoint, MCP), **Custom** (your OpenAPI spec).
- **Save** stores the agent in the project. After that you call it with the Foundry SDK or the Responses endpoint in unit 08.
