# 05 Add Agents to a Workflow

[Add Agents to a Workflow](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/5-add-agents-to-workflow?pivots=text)

- Add an **Invoke agent** node. Point it at an existing agent, or create one in the designer. The editor sets tools, knowledge, memory, and guardrails.
- The node receives user input and earlier variables, and returns text or JSON. A JSON schema on the **Details** tab is what a later If/Else can branch on.
- Store the output in a variable from **Action settings**. One categorization agent can be reused by many workflows. A different agent handles the resolution.

## Triage-Agent in the exercise

The exercise creates `Triage-Agent` from the **Invoke agent** node. Its instructions pick exactly one category, Billing, Technical, or General, with a confidence from 0 to 1. Invoice exports count as General, because Billing means money was charged, refunded, or paid wrong.

**Node settings** wire it into the flow:

```text
Input message:                  Local.CurrentTicket
Save agent output message as:   TriageOutputText
Save the output json_object as: TriageOutputJson
```

**Parameters > Text format > JSON Schema** makes the reply match this schema. A later node reads `Local.TriageOutputJson.confidence`.

```json
{
  "name": "category_response",
  "schema": {
    "type": "object",
    "properties": {
      "customer_issue": {"type": "string"},
      "category": {"type": "string"},
      "confidence": {"type": "number"}
    },
    "additionalProperties": false,
    "required": ["customer_issue", "category", "confidence"]
  },
  "strict": true
}
```
