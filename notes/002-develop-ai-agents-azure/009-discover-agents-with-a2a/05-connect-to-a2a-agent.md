# 05 Connect to your A2A agent

[Connect to your A2A agent](https://learn.microsoft.com/en-us/training/modules/discover-agents-with-a2a/5-connect-to-a2a-agent?pivots=text)

- The client needs the server base URL, then fetches the Agent Card, then sends messages.
- Non-streaming waits for one reply. Streaming returns chunks. Use streaming for a long task.
- A message has a role (user) and content, and a unique id.
- The reply is either a direct message or a task. A task means you poll or follow the stream. Advanced agents run more than one task at a time.
