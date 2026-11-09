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

The proxy forwards a Teams message to the Foundry agent's Responses endpoint. Copy the complete URL from the agent's **Details** tab, including any API-version query the portal provides; do not substitute the project endpoint. This example assumes the endpoint has **AzureRbac/Entra** authorization configured and the caller has **Foundry Agent Consumer** or another role with invocation permission. See [the endpoint's authorization schemes](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/configure-agent).

Install `requests`, `azure-identity`, and `python-dotenv`, sign in with `az login`, and put the copied URL in `.env` as `FOUNDRY_AGENT_RESPONSES_URL`. Save this illustrative helper as `invoke-agent.py`:

```python
import os

import requests
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()

def ask_foundry_agent(text: str) -> dict:
    with DefaultAzureCredential() as credential:
        token = credential.get_token("https://ai.azure.com/.default").token
        response = requests.post(
            os.environ["FOUNDRY_AGENT_RESPONSES_URL"],
            headers={"Authorization": f"Bearer {token}"},
            json={"input": text},
            timeout=60,
        )
    response.raise_for_status()
    return response.json()

print(ask_foundry_agent("What are the laptop password requirements?"))
```

`get_token` authenticates the caller; `input` carries the user's message; `raise_for_status` exposes a failed request instead of treating the error body as an answer. Run `python invoke-agent.py` and inspect the returned response before connecting this helper to a Teams message handler.

This helper sends one turn and prints the response JSON. A real proxy must also map responses to Teams activities, handle user identity, and keep each user's conversation separate. The legacy [Agent Application endpoint](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/agent-applications) supports stateless Responses calls; current endpoints can support conversation and runtime sessions, as the [hosted-session guide](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/manage-hosted-sessions) explains. Follow the contract for the URL you copied rather than assuming all Responses endpoints share the same state behavior.
