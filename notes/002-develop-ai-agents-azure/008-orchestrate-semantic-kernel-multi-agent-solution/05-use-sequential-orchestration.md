# 05 Use sequential orchestration

[Use sequential orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/5-use-sequential-orchestration?pivots=text)

- A fixed pipeline. Agent A’s output is agent B’s input. Agents do not choose the next step.
- Use it for draft then review then polish, or any stage that needs the previous stage’s data.
- Skip it when the stages are independent, one agent can do the whole job, a bad early stage should stop the rest, or the route depends on intermediate results.
- `SequentialBuilder().participants([...]).build()`, then `run_stream`. Read `WorkflowOutputEvent` for the full conversation.
