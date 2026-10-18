# 02 Understand the Microsoft Agent Framework

[Understand the Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/2-understand-agent-framework?pivots=text)

- Open-source SDK. Agents reason with a model, tools, and history. Orchestration is how several agents share one job.
- Chat clients hide the provider (`BaseChatClient`: Azure OpenAI, OpenAI, Anthropic, and others). Swap the client, keep the agent.
- Tools are custom functions or hosted tools (code interpreter, file search, web search).
- `AgentSession` stores the thread. Message roles are user, assistant, system, and tool.
- Same SDK runs a Foundry agent, a multi-agent workflow, or a human in the loop.
