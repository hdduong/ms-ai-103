# 04 Use Microsoft 365 Agents Toolkit

[Advanced — Use Microsoft 365 Agents Toolkit](https://learn.microsoft.com/en-us/training/modules/integrate-foundry-agent-with-m365/4-agents-toolkit-advanced?pivots=text)

- Skip this unit until you need custom SSO, middleware, a separate dev/stage/prod deploy, or CI. Direct portal publish covers the normal case.
- The toolkit is a VS Code or Visual Studio extension. The project type is **Custom Engine Agent**. You point it at the Foundry agent endpoint.
- Shape: Teams or Copilot, then your proxy, then the Foundry agent. The proxy is where SSO and middleware live.
- Local test is the **Microsoft 365 Agents Playground**. Then the toolkit provisions Azure, deploys the proxy, and registers the app in Teams.

| | Portal publish | Agents Toolkit proxy |
| --- | --- | --- |
| Time | Minutes | Hours to days |
| Code | None | A proxy app |
| Customization | Limited | SSO, middleware, environments |
| Debug | Foundry playground | The IDE |
