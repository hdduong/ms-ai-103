# 03 Identify Workflow Patterns

[Identify Workflow Patterns](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/3-identify-workflow-patterns?pivots=text)

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Sequential | Each node runs in order and passes its output on | A fixed pipeline: validate, enrich, answer |
| Human-in-the-loop | The flow asks a question and waits | Approval, a missing fact, a confirmation |
| Group chat | Control moves between agents from context | Several specialists on one request |

Pick the pattern before you add nodes. Sequential is the one to learn first because the path is visible.

## The exercise workflow, node by node

Lab: [Build a workflow in Microsoft Foundry](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html). It is sequential: one ticket at a time, classify, then two If/Else branches. Each line is a node and the value typed into it.

```text
Set variable   Local.SupportTickets = [ three ticket strings ]
For each       items: Local.SupportTickets        loop value: Local.CurrentTicket
  Invoke agent   Triage-Agent    input: Local.CurrentTicket
                 text -> Local.TriageOutputText   json -> Local.TriageOutputJson
  If/Else        Local.TriageOutputJson.confidence > 0.6
    If:
      If/Else      Local.TriageOutputJson.category = "Billing"
        If:   Deliver a message   "Escalate billing issue to human support team."
        Else: Invoke agent        Resolution-Agent   input: Local.TriageOutputText
                                  text -> Local.ResolutionOutputText
    Else:
      Deliver a message   "...low confidence. Requesting more details: {Local.CurrentTicket}"
```

Human-in-the-loop would swap a **Deliver a message** for an **Ask a question** node that waits for the reply. Group chat lets the agents pass control instead of following this fixed path.
