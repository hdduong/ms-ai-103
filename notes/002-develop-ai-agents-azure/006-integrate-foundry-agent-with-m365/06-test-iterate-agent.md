# 06 Test and iterate your integrated agent

[Test and iterate your integrated agent](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/6-test-iterate-agent?pivots=text)

- The Foundry playground is not the published app. Retest in Teams: the UI, sign-in, latency, and the published identity's roles. Try desktop, web, and mobile, and have someone else phrase the question.
- No reply in Teams: Bot Service missing or misconfigured, or the zip was not uploaded. Check the Bot Service resource and its logs.
- Tools work in Foundry and fail in Teams: check which runtime identity made the failing call and whether it has RBAC on the tool's resource. Also compare the selected agent version and tool configuration.
- Users cannot find it: shared scope needs the link, organization scope needs admin approval, or the tenant blocks custom apps.
- Slow replies: long instructions, a tool over a large index, or the network. Watch Foundry metrics (volume, latency, errors, tool calls) and Application Insights traces.
- Fix it in Foundry and retest the intended version. Check whether the endpoint follows the latest version or is pinned; republish channel/package changes as required. Organization distribution may need approval again.

The Foundry metrics are on the agent's **Monitor** tab.

![it-support-agent Monitor: estimated cost, token usage, evaluations, scheduled red teaming, agent runs, and Gen AI errors.](images/06-agent-monitor.png)

## Compare the same grounded answer across channels

Use the `enterprise-knowledge-agent` and attached files from [unit 03](03-publish-agent-foundry-portal.md#lab-publish-enterprise-knowledge-agent). The [official exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/05a-m365-teams-integration.html) supplies both the test prompts and the policy documents. Ask each question in a fresh Foundry conversation, then in Teams and, if licensed, Copilot.

| Prompt | Evidence expected from the exercise's sample files |
| --- | --- |
| What are the password requirements for my laptop? | IT policy: at least 12 characters, with uppercase, lowercase, numbers, and special characters |
| What are the core hours for remote employees? | Remote-work policy: Monday–Friday, 9 AM–3 PM in the employee's local time zone |
| What encryption is required on company laptops? | IT policy: BitLocker full-disk encryption |
| What MFA methods are supported for company systems? | IT policy: Authenticator, SMS backup, security keys, and phone calls |

These are facts from the fictional Contoso documents, not recommendations for your own organization's security policy. Check the cited passage as well as the answer: correct wording without retrieval evidence may come from model knowledge.

If all channels retrieve the same facts, the example demonstrates that grounding survived publication. If only Teams fails, investigate channel configuration, sign-in, runtime identity, and version routing. If every channel gives a generic answer, check that both files finished indexing and that the file-search tool is attached.

After testing, remove the personal test installation and any dedicated agent/Bot Service resources you no longer need. Check each resource's ownership before cleanup; deleting a shared resource can affect other agents.
