# 03 Options for implementing custom tools

[Options for implementing custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/3-custom-tool-options?pivots=text)

- **Function calling.** You describe the function. The agent returns the name and arguments. Your code runs it. Any language.
- **Azure Functions.** Event-driven. A trigger starts the function. Bindings connect the input and output, often storage queues.
- **OpenAPI 3.0.** Point the agent at an HTTP API spec. Auth is anonymous, API key, or managed identity.
- **Logic Apps.** Low-code workflow when you already connect apps that way.
- Use more than one on the same agent when the job needs it.
