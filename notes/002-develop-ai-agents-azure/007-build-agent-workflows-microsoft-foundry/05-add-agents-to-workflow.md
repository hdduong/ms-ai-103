# 05 Add agents to a workflow

[Add Agents to a Workflow](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/5-add-agents-to-workflow?pivots=text)

An **Invoke agent** node gives a model a defined role and maps its output into workflow variables. Instructions describe what the agent should decide; the node settings specify where its input comes from and where its result goes.

In the [ContosoPay example](03-identify-workflow-patterns.md#the-exercise-workflow-node-by-node), one agent classifies tickets and a separate agent drafts responses. Follow the [official workflow exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/06-build-workflow-ms-foundry.html) with these settings.

## Triage-Agent in the exercise

Inside **For each**, add **Invoke > Agent > Create new agent**, name it `Triage-Agent`, and use text interaction mode if prompted. Select a deployed model available in the project. The following condensed instructions preserve the exercise's classification rules:

```text
Classify the customer's issue as exactly one of Billing, Technical, or General.
Return customer_issue, category, and confidence (a number from 0 to 1).
Billing: incorrect charges, refunds, payments, or payouts.
Technical: API errors, integration/webhook faults, or platform bugs.
General: how-to questions, feature availability, exports, reports, or navigation.
Exporting or viewing invoices is General; it is Billing only when money was
charged, refunded, or paid incorrectly.
```

Under **Parameters > Text format > JSON Schema**, enter the exercise's response format:

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

The schema enforces field names and types. It does not encode the allowed category values or the numeric range; this exercise supplies those in instructions. The model's confidence is a routing signal, not a calibrated probability of correctness.

In **Node settings**, wire the input and both output forms:

```text
Input message:                  Local.CurrentTicket
Save agent output message as:   TriageOutputText
Save the output json_object as: TriageOutputJson
```

The next condition reads a field from `Local.TriageOutputJson`; the resolution agent receives the text form in `Local.TriageOutputText`. Both come from the same classification, but serve different consumers. [Unit 06](06-apply-power-fx.md#lab-branch-conditions) connects the branches.

## Resolution-Agent

Under the **Else** branch of the category condition (the non-Billing path), add another **Invoke > Agent** node and create `Resolution-Agent`. Use these condensed instructions from the exercise:

```text
Draft a professional, friendly support response for ContosoPay from the
supplied issue and category. For Technical issues, suggest 1–2 high-level
troubleshooting steps without requesting credentials, sensitive data, or logs.
For General issues, give concise guidance. Use fewer than five sentences.
Return only the drafted response; do not include internal analysis.
```

Set its input to `Local.TriageOutputText` and save its text output as `ResolutionOutputText`. Unlike triage, this node should return ordinary text for a reader. It does not need the classification schema.

On preview, an invoice-export question should be classified as General and receive guidance. An incorrect charge with sufficient confidence should bypass this agent and display the Billing message. Checking which node ran is more informative than checking the final wording alone.
