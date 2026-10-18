# 06 Apply Power Fx in Workflows

[Apply Power Fx in Workflows](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/6-apply-power-fx?pivots=text)

- Power Fx is the expression language on conditions, variables, and loops. Excel-like.
- `Local.*` is data from this run. System variables are the conversation: last message, user, current activity.
- If/Else uses a Boolean such as `Local.Confidence > 0.8`. For Each walks a list. `ForAll` applies a formula to each row.
- Useful functions: `Upper`, `Lower`, `Len`, `If`, `Sum`, `Count`, `IsBlank`, `IsEmpty`, `Concatenate`.
