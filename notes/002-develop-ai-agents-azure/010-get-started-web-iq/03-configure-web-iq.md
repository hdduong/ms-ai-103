# 03 Configure Web IQ

[Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/get-started-web-iq/3-configure-web-iq)

Create a **Microsoft Web IQ** resource in Azure, select **Global** when prompted, then connect it in the Web IQ portal. The service requires a paid subscription; free trials and subscriptions funded through Azure credits are unsupported.

| Value | Meaning |
| --- | --- |
| Web IQ AppID | Selects the application profile, enabled services, limits, and credentials. |
| Entra application/client ID | Identifies an application registered in Microsoft Entra ID. |
| API key | A secret that authenticates requests for its Web IQ profile. |

For key authentication, clients use `x-apikey`. The key selects its profile, so do not add an `APPID` query parameter. With Entra authentication, an application bound to multiple profiles must specify the intended AppID.

Test a query in the Web IQ playground before attaching an agent. If the resource is missing, verify the selected tenant, subscription, and profile. Store secrets in the connection configuration or a secret manager.

**Exam cue:** creating the Azure resource, selecting the Web IQ profile, and authenticating a client are separate steps.

[Back to module](README.md)
