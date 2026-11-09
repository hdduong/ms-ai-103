# 03 Understand agent orchestration

[Understand agent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/3-understand-agent-orchestration?pivots=text)

- One agent is one instruction set. Orchestration splits the job so each agent has a skill, then combines the results.
- A workflow is executors plus edges. An executor is an agent or your own step. Edges are direct, conditional, switch-case, fan-out, or fan-in.
- Observe workflow start, executor progress, output, errors, and requests for human input. Workflows can checkpoint and resume; event names depend on the SDK generation shown below.
- Five patterns share the same steps: define participants, choose a builder, build the workflow, run it, and read its outputs or events.

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Concurrent | Same task, agents do not see each other | Parallel analysis, voting |
| Sequential | Output of one is the input of the next | A fixed pipeline |
| Handoff | A participant transfers control to a specialist | Expert routing |
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


The exercise's feedback pipeline demonstrates the fixed-order case. The classifier depends on the summary, and the action agent needs both. Calling all three concurrently would remove those dependencies. `output_from="all"` selects which results are exposed; it does not change the graph's execution order.

For streaming with the pinned packages, replace the run-and-print step with this function call pattern:

```python
async def observe_workflow(workflow, feedback):
    async for event in workflow.run(f"Customer feedback: {feedback}", stream=True):
        if event.type == "output":
            print(event.data)
        elif event.type == "error":
            print("Workflow error:", event.data)
```

`event.type` identifies the event, while `event.data` carries its payload. Importing the older `WorkflowOutputEvent` class into this package generation is not required.
