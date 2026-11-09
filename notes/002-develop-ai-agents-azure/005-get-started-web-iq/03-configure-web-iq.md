# 03 Configure Web IQ

[Configure Web IQ](https://learn.microsoft.com/en-us/training/modules/get-started-web-iq/3-configure-web-iq)

- Two portals. The **Azure portal** creates the **Microsoft Web IQ** resource for billing and management. The **Web IQ portal** connects it to an application profile, issues credentials, and has a playground.
- Web IQ needs a paid subscription. Free trials and Azure-credit subscriptions are not supported.
- A **Web IQ AppID** is the profile: allowed APIs, rate limits, keys, and bound Entra apps. An **Entra client ID** is a registered app. An **API key** is the secret. They are not interchangeable.
- Key auth uses the `x-apikey` header. The key already selects its profile, so do not add an `APPID` parameter. With Entra auth, an app bound to several profiles must name the AppID.
- Test a query in the Web IQ **Playground** before attaching an agent. If the resource is missing, check the tenant, subscription, and profile.

## Lab: create the resource

Use the resource setup from [Integrate web search into an agent using Web IQ](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04b-web-iq.html) to prepare the music agent introduced in [unit 02](02-explore-web-iq.md#choose-a-method-for-the-music-agent-example). This part runs in the portals; it has no application code to paste.

1. In the **Azure portal**, create a **Microsoft Web IQ** resource in the **Global** region with the paid subscription and accept its terms.
2. Open **Go to Web IQ portal**, sign in, and select that same subscription and resource. Confirm the intended AppID in the profile selector.
3. In **Playground**, choose **Web**, enter an artist query, and run it. Inspect both the JSON and formatted result. Then switch to **Images** to see how the response changes.
4. Open **Profile management** and copy the default API key for the Foundry connection in [unit 04](04-use-web-iq-foundry.md#lab-band-agent).

These objects serve different purposes:

| Object created or selected | Why the agent needs it |
| --- | --- |
| Azure Web IQ resource | Associates service usage with the subscription and billing |
| AppID/profile | Selects the enabled APIs, credentials, and limits |
| API key | Authenticates retrieval through that profile |
| Foundry connection, created next | Stores the credential so the agent can call Web IQ |

A successful playground result verifies the retrieval service and its credentials. It does not yet verify an agent integration: the agent still needs the connection attached as a tool, instructions, and a traced call.
