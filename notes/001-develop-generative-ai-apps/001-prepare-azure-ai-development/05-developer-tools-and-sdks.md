# 05 Developer tools and SDKs

[Developer tools and SDKs](https://learn.microsoft.com/en-us/training/modules/prepare-azure-ai-development/5-tools-and-sdks)

- Portal work is not the whole app. You still write, test, and deploy code.
- Visual Studio fits .NET on Windows. VS Code fits open-source languages. Both can build Azure AI apps.
- In VS Code, the **Foundry Toolkit** extension is the one for Foundry projects. It browses models, agents, connections, and vector stores, deploys models, opens playgrounds, edits declarative and hosted agents (designer and YAML), and generates integration code. On the marketplace the name is **Foundry Toolkit for VS Code**, published by Microsoft. It is the new name for AI Toolkit.

![Foundry Toolkit for VS Code on the Visual Studio Marketplace, published by Microsoft.](images/05-foundry-toolkit.png)
- GitHub is the source control and DevOps side. Visual Studio and VS Code both talk to it. GitHub Copilot is the in-editor assistant.
- Languages called out: C#, Python, Node, TypeScript, Java.

Three libraries to plan for:

- **Foundry SDK** connects to a project and to Foundry assets: agents and Foundry IQ knowledge stores.
- **OpenAI API** is for chat against Foundry models that speak OpenAI syntax.
- **Foundry Tools SDKs** (or the REST APIs) call the individual tools in the subscription.

## Use the same project in VS Code

Continue the [project exercise](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/01-Explore-ai-studio.html) after [creating the project and testing its model](03-microsoft-foundry.md#lab-create-a-project-and-test-a-model). Its prerequisites include VS Code, Git, Azure CLI, and Python 3.13.

1. Install Foundry Toolkit and sign in to the Azure tenant containing your project.
2. Select that project and choose **Set as default project**.
3. Open **Developer Tools > Build > Model playground**, select the deployment you created, and send the same prompt as in the portal.

The two playgrounds call the same deployment. Setting a default project selects the extension's working context; it does not create a new deployment or grant permission to one. Compare the selected deployment and response before moving to SDK code.
