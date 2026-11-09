# 03 Identify workflow patterns

[Identify Workflow Patterns](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/3-identify-workflow-patterns?pivots=text)

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Sequential | Nodes run in a fixed order and pass results forward | Validate, classify, then draft a response |
| Human-in-the-loop | A node asks for input and suspends until it arrives | Approval or a missing fact must be supplied |
| Group chat | A manager or agents choose the next participant | Specialists need to collaborate dynamically |

Pick the pattern before adding nodes. A sequence is useful when the workflow's structure is known even though its model-generated values vary.

## The exercise workflow, node by node

The [Microsoft Learn workflow exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html) illustrates that distinction with ContosoPay support tickets. The model classifies each issue; the workflow applies fixed routing rules to its result.

| Step | Input | What the node produces | Lesson |
| --- | --- | --- | --- |
| Set variable | Three supplied ticket strings | `Local.SupportTickets` | [04: Create the data](04-create-workflows-microsoft-foundry.md#first-nodes-in-the-exercise) |
| For each | `Local.SupportTickets` | One `Local.CurrentTicket` at a time | [04: Loop over it](04-create-workflows-microsoft-foundry.md#first-nodes-in-the-exercise) |
| Invoke `Triage-Agent` | `Local.CurrentTicket` | Text and structured JSON | [05: Define the agents](05-add-agents-to-workflow.md#triage-agent-in-the-exercise) |
| First If/Else | `Local.TriageOutputJson.confidence` | High-confidence or clarification branch | [06: Apply conditions](06-apply-power-fx.md#lab-branch-conditions) |
| Second If/Else | Category on the high-confidence branch | Billing escalation or `Resolution-Agent` | [06: Apply conditions](06-apply-power-fx.md#lab-branch-conditions) |
| Invoke `Resolution-Agent` | `Local.TriageOutputText` | Draft response for Technical/General tickets | [05: Define the agents](05-add-agents-to-workflow.md#resolution-agent) |

These nodes stay **inside the For each body** after the array is created. Each ticket follows exactly one branch. The two **Deliver a message** nodes only display clarification or escalation text; they neither wait for a reply nor create a support case. Use **Ask a question** when the workflow must actually pause for a person.

The later [Python client](08-use-workflows-in-code.md#invoke) starts this saved graph. It does not construct the nodes or decide the branches itself.
