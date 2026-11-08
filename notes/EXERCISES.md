# Exercise and code index

Reviewed **2026-11-07** against the [Microsoft Learning generative AI exercises](https://microsoftlearning.github.io/mslearn-ai-studio/) and [agent exercises](https://microsoftlearning.github.io/mslearn-ai-agents/).

The lesson pages below pair the concepts with their official exercises. Each identifies the source snapshot, setup, code or portal work, and the behavior to observe. Read the surrounding explanation before copying a snippet: some blocks are excerpts that depend on the lab's existing client, environment, files, or agent.

## Generative AI applications

| Module | Exercise lesson | What to practice |
| --- | --- | --- |
| Prepare an AI project | [Project setup](001-develop-generative-ai-apps/001-prepare-azure-ai-development/07-exercise-prepare-project.md) | Foundry project, model deployment, and development environment |
| Select and evaluate models | [Model comparison](001-develop-generative-ai-apps/002-model-catalog-evaluate/06-exercise-compare-models.md) | Compare candidates and evaluate results in the portal |
| Foundry SDK and chat | [Chat application](001-develop-generative-ai-apps/003-foundry-sdk/06-exercise-chat-app.md) | Project connection, Responses API, conversation state, and streaming |
| Generative AI tools | [Tools application](001-develop-generative-ai-apps/004-use-generative-ai-tools/07-exercise-use-tools.md) | Web/file search, brochure ingestion, and conversation context |
| Optimize model behavior | [Fine-tuning](001-develop-generative-ai-apps/005-optimize-generative-ai-model-performance/06-exercise-fine-tune.md) | Training/validation data and model customization |
| Responsible AI | [Guardrails](001-develop-generative-ai-apps/006-responsible-ai-studio/07-exercise-guardrails.md) | Content filters and observable blocked responses |

## AI agents

The rows follow the current Microsoft Learn order; folder numbers remain stable.

| Module | Exercise lesson | What to practice |
| --- | --- | --- |
| Portal and VS Code | [Build an agent](002-develop-ai-agents-azure/001-develop-ai-agents-azure-vs-code/09-exercise-agent-portal-vscode.md) | Agent configuration and testing |
| Custom tools | [Function tools](002-develop-ai-agents-azure/002-build-agent-with-custom-tools/05-exercise-custom-functions.md) | Tool schema, function dispatch, and returning tool output |
| MCP | [MCP integration](002-develop-ai-agents-azure/003-connect-agent-to-mcp-tools/05-exercise-mcp-tools.md) | Tool discovery, connections, and approvals |
| Foundry IQ | [Enterprise grounding](002-develop-ai-agents-azure/004-introduction-foundry-iq/06-exercise-foundry-iq.md) | Knowledge base connection and grounded answers |
| Web IQ | [Web grounding](002-develop-ai-agents-azure/010-get-started-web-iq/05-exercise-web-iq.md) | Portal connection, current web evidence, and trace inspection |
| Microsoft 365 | [Teams and Copilot](002-develop-ai-agents-azure/005-integrate-foundry-agent-with-m365/07-exercise-m365-integration.md) | Publishing and channel validation |
| Work IQ (optional) | [Workplace context](002-develop-ai-agents-azure/005-integrate-foundry-agent-with-m365/07b-exercise-work-iq.md) | Tenant setup, OAuth, remote MCP tools, and grounded workplace answers |
| Workflows | [Agent workflow](002-develop-ai-agents-azure/006-build-agent-workflows-microsoft-foundry/09-exercise.md) | Orchestration and running a workflow from a client |
| Agent Framework | [Single agent](002-develop-ai-agents-azure/007-develop-ai-agent-with-semantic-kernel/05-exercise.md) | Framework client, instructions, and tools |
| Multiple agents | [Orchestration](002-develop-ai-agents-azure/008-orchestrate-semantic-kernel-multi-agent-solution/09-exercise.md) | Coordinating specialist agents |
| A2A | [Remote agents](002-develop-ai-agents-azure/009-discover-agents-with-a2a/06-exercise.md) | Agent cards, servers, clients, and remote tasks |

## How to run code from a lesson

1. Open its official exercise and the pinned starter/source links. This repository contains study notes; the official lab repository supplies supporting files and assets.
2. Follow that exercise's provisioning steps. Confirm the project/resource endpoint, actual deployment name, permissions, and any required connections.
3. Use a separate Python virtual environment for each lab or compatible group of labs. Install the exercise's stated requirements; do not combine every SDK version in one environment.
4. Set the documented environment values and authenticate using the lab's instructions. Keep actual credentials out of committed files.
5. Run the example, inspect output and traces, and complete the lesson's checks. Clean up only resources you created for that exercise.

For local Python labs, activate a virtual environment using the command for your shell:

```bash
python -m venv .venv
# Bash / Azure Cloud Shell
source .venv/bin/activate
```

```powershell
# Windows PowerShell, after creating .venv
.\.venv\Scripts\Activate.ps1
```

Then use the installation command in the exercise. These activation commands are general setup guidance, not an additional requirement for portal-only exercises.

## Interpreting verification

The notes have local syntax and link checks; Azure resource creation and live model calls are not executed by this documentation review. Preview features, region availability, subscription access, and model quotas can affect whether a lab runs in your tenant. Follow the exercise's current requirements when reproducing it.

See [review findings and coverage](REVIEW.md) and [source licenses](../THIRD_PARTY_NOTICES.md).

[Back to notes](README.md)
