# 05 Set up Visual Studio Code for agent development

[Set up Visual Studio Code for agent development](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/5-set-up-vs-code?pivots=text)

- Extension name in the marketplace: **Foundry Toolkit for VS Code**. After install, its icon is on the activity bar.

![Foundry Toolkit for VS Code in the Marketplace.](../../001-develop-generative-ai-apps/001-prepare-azure-ai-development/images/05-foundry-toolkit.png)
- Connect: **Azure** icon, sign in, expand the subscription, expand **Foundry**, right-click the project, **Open in Foundry Extension**.
- The lab uses **Foundry Toolkit > Microsoft Foundry Resources > Set Default Project** instead, then opens the portal agent in Agent Builder.
- The panel then shows agents, deployments, connections, and vector stores for that project.
- An agent needs a deployment. In **Resources > Model deployments**, the **+** button asks for a deployment name, a model version, and capacity. The deployment name is what the agent YAML uses as `model.id`.
- Agents created in the portal show up here. **Save to Foundry** writes the YAML back to the project.

## Prepare the Python client

The [exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/01-build-agent-portal-and-vscode.html) calls the portal-created agent from a local program. Use Python 3.13, the version specified by the exercise. Open a working folder in VS Code, then create a virtual environment:

```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv requests
az login
```

Create `.env` in that folder. Copy the project endpoint from Foundry rather than the individual model endpoint, and use the exact saved agent name:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
AGENT_NAME=it-support-agent
```

The signed-in identity needs access to this project. The model deployment, tools, and uploaded files remain on the saved agent; they do not go in this `.env`. In [unit 08](08-test-deploy-integrate.md#lab-client-chat-with-the-portal-agent), save the completed client as `agent_client.py` beside `.env` and run it in this environment.

Before moving to code, open `it-support-agent` in the Toolkit's Agent Builder and ask about a lost or stolen device. It should use the same policy file as the portal playground. This confirms you selected the correct project and agent.
