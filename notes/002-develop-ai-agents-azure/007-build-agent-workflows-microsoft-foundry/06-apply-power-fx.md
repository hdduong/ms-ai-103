# 06 Apply Power Fx in Workflows

[Apply Power Fx in Workflows](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/6-apply-power-fx?pivots=text)

- Power Fx is the expression language on If/Else, Set variable, and For Each. `Local.*` is data this run stored. System variables are the current activity, the last message, and the user.
- If/Else branches on a formula. For Each runs the same nodes for each item, such as one pass per ticket.

## Formulas

```text
Upper(Local.Input)
Local.Confidence > 0.8
If(Local.Confidence > 0.8, "Proceed", "Escalate")
Sum(Local.ItemList, Amount)
IsBlank(Local.Input)
ForAll(Local.ItemList, Upper(Name))
```

## Lab: branch conditions

The two If/Else nodes in the [workflow lab](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html):

```text
Local.TriageOutputJson.confidence > 0.6
Local.TriageOutputJson.category = "Billing"
```

The test is strict, so a confidence of exactly 0.6 takes the low-confidence branch. The Billing branch only shows a message; no ticket or email is created.
