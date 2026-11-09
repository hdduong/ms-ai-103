# 04 Create workflows in Microsoft Foundry

[Create workflows in Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/4-create-workflows-microsoft-foundry?pivots=text)

Start with a blank canvas or a pattern. The designer does not autosave; **Save** keeps a version. Variables connect nodes: one node stores a value and a later node reads it.

| Node family | What it does |
| --- | --- |
| Invoke | Call an agent and collect text or structured output |
| Flow | If/Else, Go To, For Each |
| Data | Set/reset variables and parse values |
| Basic chat | Deliver a message or ask a question |
| End | Stop the flow and optionally return a result |

The [designer migration notice](02-understand-workflows.md) applies to this portal-based example.

## First nodes in the exercise

In a Foundry project, open **Build > Agents > Workflows > Create > Blank workflow**. Save it as `ContosoPay-Customer-Support-Triage`, the name used by the [official exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html) and the later client.

Add **Data transformation > Set variable**, name the variable `SupportTickets`, and paste the following into **To value**:

```json
[
  "The API returns a 403 error when creating invoices, but our API key hasn't changed.",
  "Is there a way to export all invoices as a CSV?",
  "I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?"
]
```

This stores an array in `Local.SupportTickets`. It gives the workflow three distinct cases: an API fault, a how-to question, and an incorrect charge.

Below that node, add **Flow > For each** with these values:

```text
Select the items to loop for each: Local.SupportTickets
Loop Value Variable:              CurrentTicket
```

`Local.SupportTickets` is the complete list; `Local.CurrentTicket` is the single string for the current pass. Put the [agent nodes from unit 05](05-add-agents-to-workflow.md) and the [conditions from unit 06](06-apply-power-fx.md) inside this loop, so classification and routing happen separately for each ticket. Save after connecting the nodes.
