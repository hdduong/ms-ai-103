# 08 Use Magentic orchestration

[Use Magentic orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/8-use-magentic-orchestration?pivots=text)

- A Magentic manager plans, picks the next specialist, and keeps a task ledger of goals and steps. The plan is part of the result, so a person can read it.
- Use it when the path is not known, several specialists plus tools must shape the answer, and you want the plan written down.
- Skip it when the path is fixed, you do not need a ledger, a lighter pattern works, or you need speed. Planning is slower, and a stall loop is a real failure.
- `MagenticBuilder` plus a standard manager: max rounds, stall count, reset count. Run with `run_stream` and read `WorkflowOutputEvent`.
