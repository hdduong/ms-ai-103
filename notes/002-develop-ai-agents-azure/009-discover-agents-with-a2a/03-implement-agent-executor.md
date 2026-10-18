# 03 Implement an agent executor

[Implement an agent executor](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/3-implement-agent-executor?pivots=text)

- `AgentExecutor` is the protocol side of your logic. It does not decide discovery. The card does that.
- `execute` reads `RequestContext`, runs your code, and puts messages, task updates, or artifacts on the `EventQueue`.
- `cancel` stops a running task. A simple agent can refuse it.
- Flow: request comes in, executor calls your helper, the result becomes an event, the server sends the event back.
