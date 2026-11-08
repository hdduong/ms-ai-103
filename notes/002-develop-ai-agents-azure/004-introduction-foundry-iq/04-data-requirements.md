# 04 Configure data sources for knowledge bases

[Configure data sources for knowledge bases](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/4-data-requirements?pivots=text)

The Learn unit describes the following source options. A knowledge base can combine compatible sources; availability depends on the service version, region, and source prerequisites.

| Source | Access | Pick it when |
| --- | --- | --- |
| Azure AI Search index | Indexed | Reuse an existing searchable index, semantic configuration, and supported filters |
| Azure Blob Storage | Managed ingestion | Files in a container; the knowledge source creates and manages supporting Search indexing resources |
| Web (Bing) | Real time | Public, changing facts. You do not control the sites |
| SharePoint remote | Real time | Live SharePoint, current permissions, no index |
| SharePoint indexed | Indexed | SharePoint plus custom analyzers, enrichment, or a mix with other sources |
| OneLake | Managed ingestion | Unstructured files in a Fabric lakehouse, subject to connector and indexing requirements |

- Remote SharePoint queries the source with its access controls; indexed SharePoint depends on ingestion freshness. Actual latency and update visibility depend on the source and configuration, so “always current” or “always faster” is too absolute.
- Use web as a backup when the internal base has no answer. Do not use it when the source of the answer has to be yours.

**Add sources** on a knowledge base picks the source type. This project's menu shows Work IQ, Fabric IQ (OneLake), Blob, Web, and Azure AI Search Index, plus **Browse more**.

![knowledgebase301, Add sources: Work IQ, Fabric IQ, Azure Blob Storage, Web, Azure AI Search Index, Use existing sources, Browse more.](images/04-add-sources.png)

The exercise uses Blob as the source. The knowledge source settings from that lab:

```text
Name: ks-contosoproducts
Source: Azure Blob Storage
Container: contosoproducts
Content extraction mode: minimal
Embedding model: text-embedding-3-small
```

The current [Blob knowledge-source documentation](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-blob) lists the generated index, data source, skillset, and indexer. The [retrieval documentation](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-retrieve) also notes that scoring profiles are not applied to agentic retrieval result ranking; an existing index's scoring profile is not a reason to assume that behavior carries over.

See [the complete exercise](06-exercise-foundry-iq.md) for its key-based portal connection and ingestion checks. Screenshots preserve the original project example; generated names and portal labels can differ in your environment.
