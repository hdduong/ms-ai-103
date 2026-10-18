# 03 Publish an agent from Foundry portal to Teams

[Publish an agent from Foundry portal to Teams](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/3-publish-agent-foundry-portal?pivots=text)

- Test in the playground first. Register the `Microsoft.BotService` resource provider. Do not put secrets in the store metadata.
- Publish > Publish to Teams and Microsoft 365 Copilot. Create the Bot Service. Fill name, description, 32×32 and 192×192 PNG icons, publisher, privacy URL, and terms URL.
- Pick Shared or Organization. Prepare Agent builds the package. Upload the zip in Teams (Apps > Manage your apps > Upload a custom app) or continue in-product.
- Organization scope: Microsoft 365 admin center > Agents > Requested > Approve request and activate. App policies still limit who sees it.
- After publish, assign RBAC to the new agent identity or tools that worked in the playground fail in Teams.
- A change in Foundry is not live in Teams until you republish. Organization updates may need approval again.
