# 08 Test, deploy, and integrate agents

[Test, deploy, and integrate agents](https://learn.microsoft.com/en-us/training/modules/develop-ai-agents-azure-vs-code/8-test-deploy-integrate?pivots=text)

- Playground tests: happy path, edge cases, out-of-scope requests, multi-turn context, and that the right tool runs.
- Save in the portal, or **Save to Foundry** in VS Code. Hosted agents deploy as ZIP code or a container image (BYOI) from Container Registry.
- Creating an agent gives a stable endpoint, its own Entra identity, and a blueprint. Callers need the **Foundry User** role on the agent. API keys are not supported.
- Project RBAC does not move to the agent identity. Grant the agent identity access to every tool resource or published calls fail with authorization errors.
- Endpoint stays the same when you publish a new version. Protocol is Responses. The published endpoint is stateless, so the client keeps the history.
- Token scope is `https://ai.azure.com`. A 403 means the caller is missing Foundry User on the agent.
- Production: Application Insights, least privilege, token and rate limits, retries with backoff.

```text
POST https://{resource}.services.ai.azure.com/api/projects/{project}/agents/{agent}/endpoint/protocols/openai/responses?api-version=2025-11-15-preview
Authorization: Bearer {token}
{"input":"Say hello"}
```
