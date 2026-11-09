# 03 Configure Web IQ

[Configure Web IQ](https://learn.microsoft.com/en-us/training/modules/get-started-web-iq/3-configure-web-iq)

- Two portals. The **Azure portal** creates the **Microsoft Web IQ** resource for billing and management. The **Web IQ portal** connects it to an application profile, issues credentials, and has a playground.
- Web IQ needs a paid subscription. Free trials and Azure-credit subscriptions are not supported.
- A **Web IQ AppID** is the profile: allowed APIs, rate limits, keys, and bound Entra apps. An **Entra client ID** is a registered app. An **API key** is the secret. They are not interchangeable.
- Key auth uses the `x-apikey` header. The key already selects its profile, so do not add an `APPID` parameter. With Entra auth, an app bound to several profiles must name the AppID.
- Test a query in the Web IQ **Playground** before attaching an agent. If the resource is missing, check the tenant, subscription, and profile.

## Lab: create the resource

Lab: [Integrate web search into an agent using Web IQ](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04b-web-iq.html). Portal only.

```text
Azure portal   Create > Microsoft Web IQ, region Global, accept the terms
Web IQ portal  Get Started > sign in > pick the subscription and the new resource > Finish
               The new AppID shows in the top-left selector
               Playground: run a test query
               Profile management: copy the default API key for the Foundry connection
```

The agent side is in [04](04-use-web-iq-foundry.md#lab-band-agent).
