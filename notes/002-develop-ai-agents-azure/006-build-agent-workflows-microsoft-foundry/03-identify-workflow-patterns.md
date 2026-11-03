# 03 Identify Workflow Patterns

[Identify Workflow Patterns](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/3-identify-workflow-patterns?pivots=text)

| Pattern | What happens | Use it when |
| --- | --- | --- |
| Sequential | Each node runs in order and passes its output on | A fixed pipeline: validate, enrich, answer |
| Human-in-the-loop | The flow asks a question and waits | Approval, a missing fact, a confirmation |
| Group chat | Control moves between agents from context | Several specialists on one request |

Pick the pattern before you add nodes. Sequential is the one to learn first because the path is visible.
