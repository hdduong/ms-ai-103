# 07 Exercise — Publish an agent to Teams and Microsoft 365 Copilot

[Current exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05a-m365-teams-integration.html) · [Pinned instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/05a-m365-teams-integration.md)

Reviewed **2026-11-07**. This exercise is **portal-based**: it creates a grounded policy agent and publishes it through the unified Teams / Microsoft 365 Copilot flow. The active instructions contain prompts and configuration, **no Python implementation**. The repository also has auxiliary Python deployment/concept demos; those aren't steps in this exercise and aren't a replacement for the current publishing flow.

## Prerequisites and setup

- A Foundry project, a deployed model, and permission to create the publishing resources.
- A Microsoft 365 account with Teams access and tenant policy that permits this custom app.
- A Microsoft 365 Copilot license for the optional Copilot part. The exercise says Teams works with standard Microsoft 365 accounts.
- The publishing unit calls the roles **Azure AI Project Manager** and **Azure AI User**. The newer endpoint unit calls the invocation role **Foundry User**. Check the role and scope shown by your current portal; project access and published-agent access are separate.

Create `enterprise-knowledge-agent` in the **new Foundry** experience. Upload these two sample documents through the agent's **Upload files** control, then attach them:

| Sample document | Source |
| --- | --- |
| IT security policy | [it_security_policy.txt](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/05a-m365-teams-integration/Python/sample_documents/it_security_policy.txt) |
| Remote work policy | [remote_work_policy.txt](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/05a-m365-teams-integration/Python/sample_documents/remote_work_policy.txt) |

Set instructions based on the exercise:

```text
You are an Enterprise Knowledge Assistant for Contoso Corporation.
Answer questions about company policies and procedures using the uploaded documents.
Be professional, helpful, and concise.
Always cite your sources when referencing specific policies.
If the documents don't answer the question, say so and suggest who to contact.
```

These are fictional policies for the lab. Their password rules and office hours aren't general Microsoft recommendations or your employer's policies.

## Publish and verify

1. Test the agent in the Foundry playground and save it.
2. Open **Publish > Publish to Teams and Microsoft 365 Copilot**. Enter the app name, descriptions, publisher details, website, privacy URL, and terms URL.
3. Supply a **192×192 PNG color icon** and **32×32 PNG outline icon**. The outline icon has a transparent background.
4. Select the app scope, such as **Personal** chat. This is separate from the distribution scope below.
5. Select **Prepare Agent**, then continue the in-product publishing flow. **Individual / Shared** distribution appears under **Your agents**; **Organization** distribution appears under **Built by your org** after administrator approval and remains subject to tenant policies.
6. If the direct flow fails and custom-app uploads are permitted, use **Download & customize** and upload the generated app package in Teams.
7. Retest in Teams and, if licensed, Microsoft 365 Copilot. A successful playground run doesn't verify the published identity's permissions.

The portal creates the bot resource, manifest, icon/configuration package, and publishing configuration. You don't need to build an Agents Toolkit proxy to complete this exercise. Unit 04 covers that separate advanced option.

## Prompts and evidence to inspect

```text
What are the password requirements for my laptop?
What are the core hours for remote employees?
What encryption is required on company laptops?
What MFA methods are supported for company systems?
```

Compare each answer and citation against the uploaded file. Ask the same question in Foundry and Teams to check grounding, sign-in, latency, and identity permissions. In Copilot, select the agent or use its supported mention flow. If the answer fails only after publishing, inspect resource access for the published agent identity and the channel's logs.

## Cleanup

Delete the lab agent and review the associated bot/publishing resources in Azure. Uninstall the app from Teams; remove the Copilot extension or ask the administrator to remove an organization-catalog entry if needed. Delete a dedicated lab resource group only after reviewing its contents. A reused resource group or tenant app may contain resources other agents need.

## Remember for AI-103

- **Save** updates the agent in Foundry; **publish** distributes access to consumers.
- App scope (personal/team) and distribution scope (shared/organization) solve different problems.
- The published identity needs its own resource permissions.
- Teams/Copilot publishing is different from using **Work IQ** to read workplace data. Complete the [optional Work IQ lab](07b-exercise-work-iq.md) for that direction of integration.

Source prompts and configuration are adapted from MicrosoftLearning/mslearn-ai-agents at `4d6a32629722927bd5f1034e0b5396e0da4e962c` (2026-11-04). See [third-party notices](../../../THIRD_PARTY_NOTICES.md).
