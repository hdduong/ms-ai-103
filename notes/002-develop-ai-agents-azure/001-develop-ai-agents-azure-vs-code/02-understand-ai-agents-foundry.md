# 02 Understand AI agents and Microsoft Foundry Agent Service

[Understand AI agents and Microsoft Foundry Agent Service](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/2-understand-ai-agents-foundry?pivots=text)

- An agent uses a model plus tools to decide and act. A chat model only writes text.
- Useful because it automates routine work, scales without adding people, and stays available. Examples on the exam: personal productivity, research, sales, customer service, developer agents.
- Security risks: data leakage, prompt injection, privilege escalation, data poisoning, a compromised plugin, an action nobody reviewed, missing logs, model inversion.
- Mitigations: RBAC and least privilege, prompt filtering, human approval for sensitive actions, logs, audit of third-party tools, retrain when data drifts.
- Foundry Agent Service is managed. You do not run the compute. It calls tools, stores conversation state, and applies content filters.
- **Declarative** agents are configuration. A prompt agent is one model, instructions, and tools. A workflow agent is multi-agent YAML. A **hosted** agent is your container, and Foundry runs it. This module is the prompt agent.
