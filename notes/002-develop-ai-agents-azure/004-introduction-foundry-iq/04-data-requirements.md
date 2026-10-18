# 04 Configure data sources for knowledge bases

[Configure data sources for knowledge bases](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/4-data-requirements?pivots=text)

- Six sources. Real-time stays current. Indexed is faster and can use custom search pipelines.

| Source | Access | Use it when |
| --- | --- | --- |
| Azure AI Search index | Indexed | You already have an index, semantic ranker, or scoring profile |
| Blob Storage | Direct | PDF, docx, txt, md, HTML in a container |
| Web (Bing) | Real-time | Public, changing facts. You do not control the sites |
| SharePoint remote | Real-time | Live files, SharePoint permissions, no index to maintain |
| SharePoint indexed | Indexed | Custom analyzers or a mix with other sources |
| OneLake | Direct | Fabric lakehouse files, reports, research |

- One knowledge base can mix sources. Prefer an internal source when the answer must be checked. Use the web only as a backup.
