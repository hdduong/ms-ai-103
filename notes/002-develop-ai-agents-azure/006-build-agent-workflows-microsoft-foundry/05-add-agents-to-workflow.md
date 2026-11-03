# 05 Add Agents to a Workflow

[Add Agents to a Workflow](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/5-add-agents-to-workflow?pivots=text)

- Add an **Invoke agent** node. Point it at an existing agent, or create one in the designer. The editor sets tools, knowledge, memory, and guardrails.
- The node receives user input and earlier variables, and returns text or JSON. A JSON schema on the **Details** tab is what a later If/Else can branch on.
- Store the output in a variable from **Action settings**. One categorization agent can be reused by many workflows. A different agent handles the resolution.
