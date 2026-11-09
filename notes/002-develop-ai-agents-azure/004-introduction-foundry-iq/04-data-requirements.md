# 04 Configure data sources for knowledge bases

[Configure data sources for knowledge bases](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/4-data-requirements?pivots=text)

Six sources. One knowledge base can mix them.

| Source | Access | Pick it when |
| --- | --- | --- |
| Azure AI Search index | Indexed | You already have an index, semantic rank, filters, or a scoring profile |
| Azure Blob Storage | Direct | Files in a container: PDF, docx, txt, md, HTML. No index to maintain |
| Web (Bing) | Real time | Public, changing facts. You do not control the sites |
| SharePoint remote | Real time | Live SharePoint, current permissions, no index |
| SharePoint indexed | Indexed | SharePoint plus custom analyzers, enrichment, or a mix with other sources |
| OneLake | Direct | Unstructured files in a Fabric lakehouse |

- Remote SharePoint is slower and always current. Indexed SharePoint is faster and only as fresh as the last index.
- Use web as a backup when the internal base has no answer. Do not use it when the source of the answer has to be yours.

**Add sources** on a knowledge base picks the source type. This project's menu shows Work IQ, Fabric IQ (OneLake), Blob, Web, and Azure AI Search Index, plus **Browse more**.

![knowledgebase301, Add sources: Work IQ, Fabric IQ, Azure Blob Storage, Web, Azure AI Search Index, Use existing sources, Browse more.](images/04-add-sources.png)

## Lab: Blob source for product-expert-agent

Lab: [Integrate an AI agent with Foundry IQ](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04-integrate-agent-with-foundry-iq.html). Portal steps:

1. Create `product-expert-agent`. Under **Knowledge > Add > Connect to Foundry IQ**, create or pick an Azure AI Search resource in the same resource group and region (Free if offered, else Basic).
2. Create a Storage account (Standard, LRS) and a container `contosoproducts`. Upload the three PDFs from the lab's [contoso-products.zip](https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip).
3. On the Search resource, **Keys > API Access control > Both**, so the lab's key connection works.
4. Back in **Knowledge**, create a knowledge base and add the Blob source. Wait until the source is active, then attach the base to the agent.

```text
Name: ks-contosoproducts
Description: Contoso product catalog items
Source: Azure Blob Storage
Container: contosoproducts
Authentication: API key
Content extraction mode: minimal
Embedding model: text-embedding-3-small
Chat completions model: gpt-5
```

Blob is managed ingestion: the source creates the Search index, data source, skillset, and indexer for you. The agent's instructions and the client are in [05](05-configure-retrieval.md#lab-ask-then-approve-the-knowledge-base-call).
