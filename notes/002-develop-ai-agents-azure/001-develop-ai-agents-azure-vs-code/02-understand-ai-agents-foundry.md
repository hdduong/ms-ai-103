# 02 Understand AI agents and Microsoft Foundry Agent Service

[Understand AI agents and Microsoft Foundry Agent Service](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/2-understand-ai-agents-foundry?pivots=text)

- An agent combines a model, instructions, tools, and orchestration to carry out a task. A model can produce text or tool-call requests; the surrounding application/service executes actions.
- Useful because it automates routine work, scales without adding people, and stays available. Examples on the exam: personal productivity, research, sales, customer service, developer agents.
- Security risks: data leakage, prompt injection, privilege escalation, data poisoning, a compromised plugin, an action nobody reviewed, missing logs, model inversion.
- Mitigations: RBAC and least privilege, prompt filtering, human approval for sensitive actions, logs, audit of third-party tools, retrain when data drifts.
- Foundry Agent Service is managed. You do not run the compute. It integrates tools, can store conversation state, and applies configured content-safety controls. Hosted tools run in the service; local function tools still need client execution.
- **Declarative** agents are configuration. A prompt agent is one model, instructions, and tools. A workflow agent is multi-agent YAML. A **hosted** agent is your container, and Foundry runs it. This module is the prompt agent.

![Build > Agents. Both agents are Prompt type. Workflows has its own tab.](images/02-agents-list.png)

The module exercise calls the saved agent by name. The model stays on the agent.

```python
response = openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    input="",
)
print(response.output_text)
```
