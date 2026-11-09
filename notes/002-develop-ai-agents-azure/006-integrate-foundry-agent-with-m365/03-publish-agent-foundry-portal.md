# 03 Publish an agent from Foundry portal to Teams

[Publish an agent from Foundry portal to Teams](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/3-publish-agent-foundry-portal?pivots=text)

- Test in the playground first. Confirm the publishing and invocation roles for your agent model ([unit 02](02-understand-publishing-options.md)); Learn's older role names are **Azure AI Project Manager** and **Azure AI User**. Register the **Microsoft.BotService** resource provider.
- Metadata: display name, description, PNG icons at **32×32** and **192×192**, publisher name and contact, privacy policy URL, terms of use URL. Do not put a key in any of those fields.
- Portal path: open the agent, **Publish**, **Publish** again, **Publish to Teams and Microsoft 365 Copilot**. Create the Bot Service in the dialog. **Prepare Agent**. Download the zip, or stay in the portal flow.
- Test a downloaded zip: Teams, **Apps > Manage your apps > Upload an app > Upload a custom app**.
- Organization scope: Microsoft 365 admin center, **Agents > All > Requested**, **Approve request and activate**. It then appears under **Built by your org**. App policies still decide who can open it.
- Confirm that the agent runtime identity has RBAC on every Azure resource a tool uses. Check the active version before testing a change: the new agent model can follow the latest version or be pinned to one version. Republish changes to channel configuration or distribution as required by the portal flow; organization distribution may need approval again.

The identity to grant roles to is on the agent's **Details** tab, under **Identity & access**.

![it-support-agent Details: Entra agent identity and Entra agent blueprint. IDs are masked.](images/03-agent-identity.png)

## Lab: publish enterprise-knowledge-agent

Use [Publish an agent to Teams and Microsoft 365 Copilot](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05a-m365-teams-integration.html) to follow one agent from grounded answers to a published channel. The exercise uses the portal; it does not require the auxiliary Python applications found in the repository. Its Teams path requires an account with Teams access and permission to use custom apps; its Copilot path requires the license specified by the exercise.

### Prepare the grounded agent

Create or reuse a Foundry project with a deployed model. Download the two source documents from the [exercise's sample-data folder](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/05a-m365-teams-integration/Python/sample_documents): **it_security_policy.txt** and **remote_work_policy.txt**. Save the raw file contents, not the GitHub HTML page.

Create `enterprise-knowledge-agent`, select the model, and use **Tools > Upload files > Attach** for those documents. Set the adapted instructions:

```text
You are an Enterprise Knowledge Assistant for Contoso Corporation.
Answer questions about company policies and procedures using the uploaded documents.
Be professional, helpful, and concise.
Always cite your sources when referencing specific policies.
If the documents don't answer the question, say so and suggest who to contact.
```

The document instruction identifies the evidence source; the citation instruction lets you check the answer; the final line defines the missing-evidence behavior. These prompts do not upload or retrieve files themselves, which is why the **Attach** step matters.

Ask the following questions in the playground, inspect the file-search results and citations, then save. The [comparison table in unit 06](06-test-iterate-agent.md#compare-the-same-grounded-answer-across-channels) gives the expected facts from the sample documents.

```text
What are the password requirements for my laptop?
What are the core hours for remote employees?
What encryption is required on company laptops?
What MFA methods are supported for company systems?
```

### Publish the tested agent

1. Select **Publish > Publish to Teams and Microsoft 365 Copilot**. Supply the display name, descriptions, publisher and policy URLs, a **192×192** color PNG, and a **32×32** white-outline PNG with a transparent background. Use **Personal** app scope for this example.
2. Select **Prepare Agent**, then choose **Individual / Shared** for the exercise's personal test, or **Organization** when tenant-wide distribution and its approval are intended.
3. Complete the in-product publishing flow. If using a downloaded package, choose **Download & customize** and upload it through **Teams > Apps > Manage your apps > Upload a custom app**, subject to your tenant's app policy.
4. Open the agent in Teams and repeat the same questions. If licensed, repeat them in Microsoft 365 Copilot. Continue in [unit 06](06-test-iterate-agent.md#compare-the-same-grounded-answer-across-channels) to compare evidence and diagnose differences.

**App scope** controls where the app is used, such as a personal chat. **Distribution scope** controls who can discover/install it. **Save** retains the agent configuration; **Publish** makes the channel package available. A successful playground answer therefore does not prove that channel permissions, sign-in, or distribution are correct.
