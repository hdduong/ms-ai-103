# 05 Set up Visual Studio Code for agent development

[Set up Visual Studio Code for agent development](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/5-set-up-vs-code?pivots=text)

- Extension name in the marketplace: **Foundry Toolkit for VS Code**. After install, its icon is on the activity bar.
- Connect: **Azure** icon, sign in, expand the subscription, expand **Foundry**, right-click the project, **Open in Foundry Extension**.
- The panel then shows agents, deployments, connections, and vector stores for that project.
- An agent needs a deployment. In **Resources > Model deployments**, the **+** button asks for a deployment name, a model version, and capacity. The deployment name is what the agent YAML uses as `model.id`.
- Agents created in the portal show up here. **Save to Foundry** writes the YAML back to the project.
