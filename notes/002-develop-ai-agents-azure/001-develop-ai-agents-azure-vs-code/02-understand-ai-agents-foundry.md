# 02 Understand AI agents and Microsoft Foundry Agent Service

[Understand AI agents and Microsoft Foundry Agent Service](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/2-understand-ai-agents-foundry?pivots=text)

- An agent uses a model plus tools to decide and act. A chat model only writes text.
- Useful because it automates routine work, scales without adding people, and stays available. Examples: productivity, research, sales, customer service, developer agents.
- Security risks: data leakage, prompt injection, privilege escalation, data poisoning, supply-chain plugins, unreviewed autonomous actions, missing logs, model inversion.
- Mitigations: RBAC and least privilege, prompt filtering, human approval for sensitive actions, logs, audit of third-party tools, retrain when data drifts.
- Foundry Agent Service is managed. You do not run the compute. It calls tools for you, stores conversation state, and applies content filters.
- **Declarative** agents are configuration. Prompt-based is one model, instructions, and tools. Workflow agents are multi-agent YAML. **Hosted** agents are your container, Foundry runs it. This module is prompt-based.
