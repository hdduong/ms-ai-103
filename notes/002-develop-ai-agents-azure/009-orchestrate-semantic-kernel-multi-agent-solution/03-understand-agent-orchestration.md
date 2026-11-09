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

The Learn pages and the current SDK (`agent-framework-core` 1.12, `agent-framework-orchestrations` 1.0) name some things differently:

| Learn page | Current SDK |
| --- | --- |
| `WorkflowOutputEvent`, `RequestInfoEvent`, and the other event classes | One `WorkflowEvent`. Check `event.type`: `started`, `output`, `error`, `executor_invoked`, `executor_completed`, `request_info` |
| `chat_client.create_agent(...)` | `chat_client.as_agent(...)` or `Agent(client=...)` |
| `workflow.run_stream(...)` | `workflow.run(..., stream=True)` |
| Group chat "runs agents in parallel" | A manager picks one speaker per turn |

The [lab](05-use-sequential-orchestration.md#lab-feedback-pipeline) prints every agent's message like this, with `output_from="all"`:

```python
async def print_feedback_outputs(workflow, feedback):
    result = await workflow.run(f"Customer feedback: {feedback}")
    for response in result.get_outputs():
        for message in response.messages:
            print(message.author_name, message.text)
```
