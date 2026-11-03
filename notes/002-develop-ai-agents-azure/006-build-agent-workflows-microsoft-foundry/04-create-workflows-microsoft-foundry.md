# 04 Create workflows in Microsoft Foundry

[Create workflows in Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/4-create-workflows-microsoft-foundry?pivots=text)

- Start from a blank canvas or a pattern such as sequential. The designer does not autosave. Save to keep a version.
- You chat with the workflow in the same window to see each node run.

| Node | What it does |
| --- | --- |
| Invoke | Calls an agent in the project, or creates one. Text or JSON |
| Flow | If/Else, Go To, For Each |
| Data | Set variable, reset variable, parse a value |
| Basic chat | A message or a question to the user |
| End | Stops the flow and can return a result |

Variables are the shared state. An agent result or a user answer becomes the input of a later node.
