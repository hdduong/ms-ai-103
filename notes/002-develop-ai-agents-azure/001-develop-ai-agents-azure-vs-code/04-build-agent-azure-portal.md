# 04 Build your first agent in Microsoft Foundry

[Build your first agent in Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/4-build-agent-azure-portal?pivots=text)

- Portal path: `https://ai.azure.com`, open the project, **Build > Agents**, **Create**.
- You set the name, a description, and a **deployed** model. The dropdown only lists deployments in this project.
- Instructions are the system prompt. Temperature and top p are on the same screen. Unit 06 is the exam detail for those numbers.
- **Playground** keeps the session history, so you can test more than one turn before you deploy.
- **Tools** (also **Build > Tools**) has three groups: **Configured** (Code Interpreter, File Search), **Catalog** (Bing, Azure AI Search, SharePoint, MCP), **Custom** (your OpenAPI spec).
- **Save** stores the agent in the project. After that you call it with the Foundry SDK or the Responses endpoint in unit 08.

![it-support-agent in the portal: model, instructions, tools, the chat playground, YAML, and Publish.](images/04-agent-playground.png)

## Lab: it-support-agent

The [Build AI agents with portal and VS Code exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/01-build-agent-portal-and-vscode.html) applies the settings above to an IT help desk. Create/select a Foundry project in the new portal experience, create `it-support-agent`, and select its deployed model (the exercise uses the default, such as `gpt-5`). Set these adapted instructions, then **Save**:

```text
You are an IT Support Agent for Contoso Corporation.
Help employees with technical issues and IT policy questions.
Use the uploaded policy documentation for policy answers.
Be professional and helpful. If you don't know, admit it and suggest IT support.
Collect the required details before proposing a support ticket.
```

| Tool | File from the lab | Used for |
| --- | --- | --- |
| File search | [IT_Policy.txt](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt) | Policy answers |
| Code interpreter (+ Files) | [system_performance.csv](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv) | Metrics and charts |

Download the linked files and upload each to the corresponding tool. File Search indexes policy text for retrieval; Code Interpreter needs the CSV as a file in its execution environment. Uploading one file to one tool does not make it available to the other. Save the agent, wait for File Search indexing, then test both tools in the playground:

```text
What's the policy for password resets?
Analyze the system performance data and identify when CPU usage exceeded 80%.
Create a line chart showing memory usage over time.
What are the average, minimum, and maximum disk usage values?
```

A correct policy answer should use the document, and a chart prompt should produce an artifact rather than just a description of a chart. If one fails, inspect that tool's file attachment before changing the model prompt. Next, open the same agent in [VS Code](05-set-up-vs-code.md); [unit 08](08-test-deploy-integrate.md#lab-client-chat-with-the-portal-agent) reuses it from Python.
