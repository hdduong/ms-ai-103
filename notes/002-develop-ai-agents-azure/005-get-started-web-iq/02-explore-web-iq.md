# 02 Explore Web IQ

[Explore Web IQ](https://learn.microsoft.com/en-us/training/modules/get-started-web-iq/2-explore-web-iq)

- Web IQ gives an agent public-web evidence that is newer than the model's training data.
- Foundry's built-in **Web search** tool already grounds answers with citations. Pick **Web IQ** when you want more control: dedicated news, image, and video search, **Browse** for a known URL, and settings for result count, content format, and length. It needs its own Web IQ resource and connection.
- It returns query-relevant passages, so less unrelated text reaches the model. Reuse it from REST, SDKs, or MCP.
- **Search** finds a URL. **Browse** reads a URL you already have. A flow can search, pick a result, then browse it.
- **Auto** returns typed evidence across web, news, images, videos, and structured answers (weather, time zones). It is a beta REST API and needs an AppID enabled for it. The MCP tools are only `web`, `news`, `images`, `videos`, and `browse`.
- Treat results as evidence: check the source and the event date, keep the citations, and say when sources conflict.

| Method | Use it when | Example ask |
| --- | --- | --- |
| Web Search | You need to find pages | Official tour announcements for a band |
| News | You need recent reporting | Coverage of a newly announced tour |
| Images | You need pictures with source links | Photos of the band or the venue |
| Videos | You need videos with source links | Live performance clips or interviews |
| Browse | You already have the URL | Read the tour page that search found |
| Auto | You want evidence across types in one call | Announcements across web and news |

Tuning knobs, as Web IQ request settings:

```text
query and site restriction   focus on the sources that matter
language and region          match results to the user's market
result count, content length balance coverage against the context budget
content format               query-relevant passages for grounding, or full documents
```
