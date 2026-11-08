# 02 Understand the Microsoft Agent Framework

[Understand the Microsoft Agent Framework](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/2-understand-agent-framework?pivots=text)

- Open-source SDK. An agent reasons with a model, tools, and history. Orchestration is how several agents share one job. You can swap the chat client without rewriting the agents.
- Chat clients hide the provider behind `BaseChatClient`. This path uses Foundry.
- Tools are your functions plus provider tools: Code Interpreter, File Search, Web Search.
- `AgentSession` carries conversational state; provider-managed history still needs the appropriate session identifier to resume. Common message roles are user, assistant, system, and tool.
- The same agents can run under concurrent, sequential, handoff, group chat, or Magentic. The next units are those five patterns.

The multi-agent exercise uses sequential: summarize, classify, then recommend an action. The following is a helper for already-created agents. [09-exercise.md](09-exercise.md) contains the complete program and the current pinned package setup.

```python
from agent_framework.orchestrations import SequentialBuilder

async def run_feedback(summarizer_agent, classifier_agent, action_agent, feedback):
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    result = await workflow.run(f"Customer feedback: {feedback}")
    print(result.get_outputs())
```
