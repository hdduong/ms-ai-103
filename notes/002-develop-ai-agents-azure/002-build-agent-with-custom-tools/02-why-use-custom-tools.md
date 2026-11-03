# 02 Why use custom tools

[Why use custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/2-why-use-custom-tools?pivots=text)

- A custom tool is your function, your Azure Function, or your HTTP API. The agent decides when to call it from the user message.
- The exam reasons: automate a repeated step, return the same answer every time, and reach a system the model cannot see (CRM, inventory, scheduling, tickets, an LMS).
- Flow on the diagram: the user asks, the agent picks the tool, the tool returns data, the agent answers. You do not write the `if` that calls the tool.
