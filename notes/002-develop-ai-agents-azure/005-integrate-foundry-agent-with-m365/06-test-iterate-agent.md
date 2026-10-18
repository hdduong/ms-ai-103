# 06 Test and iterate your integrated agent

[Test and iterate your integrated agent](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/6-test-iterate-agent?pivots=text)

- The Foundry playground is not the Teams client. After publish, test desktop, web, and mobile, and have someone else phrase the questions.
- No reply in Teams: Bot Service missing, bad config, or the package was not uploaded. Check the bot resource and its logs.
- Tools work in Foundry and fail in Teams: the published identity has no RBAC. Grant it on the resources the tools use.
- Users cannot find it: wrong scope, approval still pending, or the tenant blocks custom apps.
- Slow replies: shorter instructions, smaller tool queries, or the network. Foundry metrics and Application Insights show volume, latency, errors, and tool calls.
- Fix the agent in Foundry, test, republish. Keep a checklist of the scenarios you run before each package.
