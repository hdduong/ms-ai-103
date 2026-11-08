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

## What the proxy calls

This is an illustrative endpoint call, not a complete Teams bot or code from the portal-only publishing exercise.

The proxy forwards each Teams message to the Foundry agent's stable Responses endpoint. It signs in with Entra. API keys do not work on this endpoint. The proxy's identity needs **Foundry User** on the agent.

```python
import requests
from azure.identity import DefaultAzureCredential

def ask_foundry_agent(text: str) -> dict:
    token = DefaultAzureCredential().get_token("https://ai.azure.com/.default").token
    url = (
        "https://{resource-name}.services.ai.azure.com/api/projects/<project-name>"
        "/agents/it-support-agent/endpoint/protocols/openai/responses"
        "?api-version=2025-11-15-preview"
    )
    response = requests.post(
        url,
        headers={"Authorization": f"Bearer {token}"},
        json={"input": text},
        timeout=60,
    )
    response.raise_for_status()
    return response.json()

print(ask_foundry_agent("What are the laptop password requirements?"))
```

The endpoint is stateless. The proxy has to keep the conversation history if Teams needs more than one turn.
