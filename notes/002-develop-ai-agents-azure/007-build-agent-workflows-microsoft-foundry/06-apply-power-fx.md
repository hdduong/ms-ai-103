# 06 Apply Power Fx in workflows

[Apply Power Fx in Workflows](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/6-apply-power-fx?pivots=text)

Power Fx expressions make deterministic decisions from workflow data. `Local.*` identifies variables saved during this run. If/Else evaluates a Boolean expression; For Each selects the collection to iterate.

## Formulas

These are independent examples for the designer's formula fields:

```text
Upper(Local.Input)
Local.Confidence > 0.8
If(Local.Confidence > 0.8, "Proceed", "Escalate")
Sum(Local.ItemList, Amount)
IsBlank(Local.Input)
ForAll(Local.ItemList, Upper(Name))
```

## Lab: branch conditions

Continue after the [triage agent](05-add-agents-to-workflow.md#triage-agent-in-the-exercise) in the [workflow exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html). Add an If/Else node inside the loop and set its condition to:

```text
Local.TriageOutputJson.confidence > 0.6
```

The `If` branch continues automated routing. The `Else` branch adds **Deliver a message** with:

```text
The support ticket classification has low confidence. Requesting more details about the issue: "{Local.CurrentTicket}"
```

The braces insert the actual ticket into the message. A confidence of exactly `0.6` goes to this branch because the comparison uses `>`.

Inside the first condition's **If** branch, add a second If/Else:

```text
Local.TriageOutputJson.category = "Billing"
```

Its **If** branch delivers `Escalate billing issue to human support team.` Its **Else** branch invokes [Resolution-Agent](05-add-agents-to-workflow.md#resolution-agent). These are fixed rules applied to a model-generated value.

| Sample triage output | Branch to expect |
| --- | --- |
| Technical, confidence 0.9 | Resolution-Agent |
| Billing, confidence 0.9 | Billing escalation message |
| Billing, confidence 0.6 | Low-confidence message |

Use known sample values in a test copy of the workflow to check these boundaries separately from model variability. The messages in this exercise only display text; they create no support ticket or email and do not wait for a user's answer. Save, preview the three-ticket run, then [invoke the same saved workflow from code](08-use-workflows-in-code.md#invoke).
