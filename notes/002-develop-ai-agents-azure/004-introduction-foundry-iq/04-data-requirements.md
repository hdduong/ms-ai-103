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

The exercise uses Blob as the source. The knowledge source settings from that lab:

```text
Name: ks-contosoproducts
Source: Azure Blob Storage
Container: contosoproducts
Content extraction mode: minimal
Embedding model: text-embedding-3-small
```
