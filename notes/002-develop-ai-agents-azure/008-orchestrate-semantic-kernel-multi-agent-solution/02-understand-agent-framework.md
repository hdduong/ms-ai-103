# 02 Understand the Microsoft Agent Framework

[Understand the Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/2-understand-agent-framework?pivots=text)

- Open-source SDK. An agent reasons with a model, tools, and history. Orchestration is how several agents share one job. You can swap the chat client without rewriting the agents.
- Chat clients hide the provider behind `BaseChatClient`. This path uses Foundry.
- Tools are your functions plus provider tools: Code Interpreter, File Search, Web Search.
- `AgentSession` keeps the thread. Message roles are USER, ASSISTANT, SYSTEM, and TOOL.
- The same agents can run under concurrent, sequential, handoff, group chat, or Magentic. The next units are those five patterns.

The multi-agent exercise uses sequential: summarize, classify, then recommend an action.

```python
from agent_framework.orchestrations import SequentialBuilder

workflow = SequentialBuilder(
    participants=[summarizer_agent, classifier_agent, action_agent],
    output_from="all",
).build()
result = await workflow.run(f"Customer feedback: {feedback}")
print(result.get_outputs())
```
