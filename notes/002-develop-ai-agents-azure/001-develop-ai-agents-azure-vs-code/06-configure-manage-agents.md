# 06 Configure and manage agents in Visual Studio Code

[Configure and manage agents in Visual Studio Code](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/6-configure-manage-agents?pivots=text)

- This YAML is for a declarative prompt agent. Hosted agents are code. Workflow agents use a different schema.
- Designer and YAML stay in sync. Name, model deployment, description, instructions, and a generated agent id.
- Temperature about 0.1–0.3 for steady answers, 0.7–1.0 for variety. Business agents often sit around 0.3–0.7. Top p default is 1.0.
- Commit the YAML. One agent, one job. Test in the playground after each change.

```yaml
version: 1.0.0
name: healthcare-assistant
description: Schedules appointments and answers policy questions
model:
  id: gpt-4.1
  options:
    temperature: 0.5
    top_p: 1
instructions: |
  You help staff schedule appointments.
  Never share patient medical information.
tools: []
```
