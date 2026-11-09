# 06 Configure and manage agents in Visual Studio Code

[Configure and manage agents in Visual Studio Code](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/6-configure-manage-agents?pivots=text)

- This YAML is a declarative prompt agent. Hosted agents are code. Workflow agents use a different schema.
- Designer and YAML stay in sync. You set the name, the model deployment, the description, the instructions, and the extension generates the agent id.
- Temperature about 0.1–0.3 for steady answers, 0.7–1.0 for variety. Business agents often sit around 0.3–0.7. Top p default is 1.0.
- Commit the YAML. One agent, one job. Test in the playground after each change.

## Prompt agent

```yaml
# yaml-language-server: $schema=https://aka.ms/ai-foundry-vsc/agent/1.0.0
version: 1.0.0
name: healthcare-assistant
description: Schedules appointments and answers policy questions
id: agent-abc123xyz
metadata:
  authors:
    - developer-name
  tags:
    - healthcare
    - scheduling
model:
  id: gpt-4.1 # deployment name
  options:
    temperature: 0.5
    top_p: 1
instructions: |
  You help staff schedule appointments.
  Never share patient medical information.
tools: []
```
