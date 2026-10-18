# 04 Use concurrent orchestration

[Use concurrent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/4-use-concurrent-orchestration?pivots=text)

- Every agent gets the same task and works alone. Combine the replies, or leave them as separate side effects.
- Use it for brainstorming, ensemble reasoning, a vote, or when waiting in series is too slow. You can call every agent or only the ones the task needs.
- Skip it when the next step needs the previous answer, the order must be stable, quota cannot cover parallel calls, agents would write the same data at once, or you cannot merge conflicts.
- `ConcurrentBuilder().participants([...]).build()`, then `run`, then `get_outputs()`. Each message has an author.
