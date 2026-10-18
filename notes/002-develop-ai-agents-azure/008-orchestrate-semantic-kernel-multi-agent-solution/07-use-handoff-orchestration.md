# 07 Use handoff orchestration

[Use handoff orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/7-use-handoff-orchestration?pivots=text)

- One agent works, then passes the conversation to a specialist. The next agent is not known at the start.
- Use it when the domain shows up in the text (billing versus technical) and you can name the handoff rule.
- Skip it when the order is fixed, a simple rule is enough, a wrong route annoys the user, work must run in parallel, or agents can bounce forever.
- Implementation is a classifier with structured JSON, then a switch-case edge. Always include a default case.
