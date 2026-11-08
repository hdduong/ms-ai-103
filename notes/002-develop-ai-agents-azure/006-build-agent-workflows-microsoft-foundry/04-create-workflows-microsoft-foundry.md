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

## First nodes in the exercise

Lab: [Build a workflow in Microsoft Foundry](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html). Start blank, **Save** it as `ContosoPay-Customer-Support-Triage`, then add these nodes.

**Set variable** (Data transformation). Name `SupportTickets`, so it shows as `Local.SupportTickets`. **To value**:

```text
[
  "The API returns a 403 error when creating invoices, but our API key hasn't changed.",
  "Is there a way to export all invoices as a CSV?",
  "I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?"
]
```

**For each** (Flow). Items and loop variable:

```text
Select the items to loop for each:  Local.SupportTickets
Loop Value Variable:                CurrentTicket
```

**Deliver a message** (Basics), on the low-confidence branch. Curly braces put a variable into the text:

```text
The support ticket classification has low confidence. Requesting more details about the issue: "{Local.CurrentTicket}"
```
