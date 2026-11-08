# 03 Understand agent orchestration

[Understand agent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/3-understand-agent-orchestration?pivots=text)

- One agent is one instruction set. Orchestration splits the job so each agent has a skill, then combines the results.
- A workflow is executors plus edges. An executor is an agent or your own step. Edges are direct, conditional, switch-case, fan-out, or fan-in.
- Events to log: `WorkflowStartedEvent`, `WorkflowOutputEvent`, `WorkflowErrorEvent`, `ExecutorInvokeEvent`, `ExecutorCompleteEvent`, `RequestInfoEvent`. Workflows can checkpoint and resume.
- Five patterns, same shape: define agents, pick a builder, optional callbacks, start the runtime, invoke, read results async.

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Concurrent | Same task, agents do not see each other | Parallel analysis, voting |
| Sequential | Output of one is the input of the next | A fixed pipeline |
| Handoff | One agent at a time, control moves by rules | Expert routing |
| Group chat | A manager picks the next speaker | Debate, maker-checker |
| Magentic | A manager writes a plan and a ledger | Open-ended work |

The exercise runs that sequential pipeline and prints every agent's message.

```python
result = await workflow.run(f"Customer feedback: {feedback}")
outputs = result.get_outputs()
for response in outputs:
    for message in response.messages:
        print(message.author_name, message.text)
```
