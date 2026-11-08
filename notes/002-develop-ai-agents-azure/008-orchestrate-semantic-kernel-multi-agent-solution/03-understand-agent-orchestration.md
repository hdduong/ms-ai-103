# 03 Understand agent orchestration

[Understand agent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/3-understand-agent-orchestration?pivots=text)

- One agent is one instruction set. Orchestration splits the job so each agent has a skill, then combines the results.
- A workflow is executors plus edges. An executor is an agent or your own step. Edges are direct, conditional, switch-case, fan-out, or fan-in.
- In `agent-framework-core==1.12.1`, workflow events use the unified `WorkflowEvent` shape: inspect `event.type` such as `started`, `output`, `error`, `executor_invoked`, `executor_completed`, or `request_info`, and read `event.data`. Older class names such as `WorkflowOutputEvent` on some Learn pages belong to earlier API shapes. Workflows also support checkpointing and resume when configured.
- Five patterns share a basic shape: define agents, configure a builder, build the workflow, await `workflow.run(...)`, and inspect outputs. Use `workflow.run(..., stream=True)` to iterate workflow events.

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Concurrent | Same task, agents do not see each other | Parallel analysis, voting |
| Sequential | Participants run in order; by default the accumulated conversation reaches the next agent | A fixed pipeline |
| Handoff | One agent at a time, with control transferred to a specialist | Expert routing |
| Group chat | A manager picks the next speaker | Debate, maker-checker |
| Magentic | A manager writes a plan and a ledger | Open-ended work |

The [complete exercise](09-exercise.md) pins `agent-framework-orchestrations==1.0.1`, runs that sequential pipeline, and prints its outputs. This helper assumes the workflow was built with `output_from="all"`:

```python
async def print_feedback_outputs(workflow, feedback):
    result = await workflow.run(f"Customer feedback: {feedback}")
    for response in result.get_outputs():
        for message in response.messages:
            print(message.author_name, message.text)
```
