# 04 Deploy models to endpoints

[Deploy models to endpoints](https://learn.microsoft.com/en-us/training/modules/model-catalog-evaluate/4-deploy-models?pivots=text)

- Deploy from **Discover → Models**. Custom settings change the deployment type and name.
- The **deployment name** is what the app passes as `model`. It can differ from the catalog ID.
- **Global Standard** is a common starting point when supported and suitable for the application's processing-location requirements. Availability and quota depend on model, region, and subscription.

![gpt-4.1 details. Deployment type GlobalStandard, provisioning Succeeded, key masked. The endpoint is the OpenAI v1 URL on the Foundry resource.](images/04-deployment-details.png)

## Deployment choice

| Processing scope | Types in the current deployment reference | Main distinction |
| --- | --- | --- |
| Global | Standard, Provisioned, Batch | Processing can use global infrastructure |
| Data Zone | Standard, Provisioned, Batch | Processing remains in the selected US, EU, or APAC zone |
| Azure geography | Standard, Regional Provisioned | Processing remains in the chosen Azure geography; no regional Batch type is listed |
| Developer | Fine-tuned model evaluation | Short-lived evaluation deployment |

- **Standard** is token-based consumption. **Provisioned** reserves throughput using PTUs.
- **Batch** offers discounted asynchronous processing, targeting completion within 24 hours; this is not a completion guarantee.
- **Developer** has a 24-hour lifetime, no availability SLA, and no data-residency guarantee. Use it for evaluation.
- Data at rest and inference processing location are separate considerations.

**Review note (2026-11-07):** The training unit describes regional processing as a single region. The current [deployment-types product reference](https://learn.microsoft.com/en-us/azure/ai-foundry/foundry-models/concepts/deployment-types) describes processing within an **Azure geography**, potentially across its regions. Use that current product reference when evaluating residency requirements, and check model-specific availability.

## After deployment

- Partner/community models may require a Marketplace agreement. Managed compute also requires VM quota and an instance count.
- Check **Succeeded** status and test in the playground. Manage deployments under **Build → Models**.
- Calling the deployment requires a compatible endpoint, authentication, and its deployment name. The direct OpenAI SDK lab uses the **Azure OpenAI `/openai/v1/` endpoint** with Microsoft Entra authentication.
- The screenshot retains the earlier `gpt-4.1` example. The [current comparison exercise](06-exercise-compare-models.md) deploys `gpt-5.2` and `gpt-5-mini`.
