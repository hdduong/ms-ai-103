# 02 Explore Web IQ

[Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/get-started-web-iq/2-explore-web-iq)

Web IQ retrieves public-web evidence for an agent. Use it when the question needs information newer than the model's training data or more control over retrieval.

| Method | Choose it when |
| --- | --- |
| Web Search | You need to discover relevant pages. |
| News | You need recent reporting. |
| Images | You need image results and their source metadata. |
| Videos | You need video results and source links. |
| Browse | You already know which URL to read. |
| Auto | You want evidence across supported answer types; this is a beta REST capability requiring an enabled AppID. |

**MCP distinction:** the documented tools are `web`, `news`, `images`, `videos`, and `browse`. Do not assume the Auto REST capability is an MCP tool or that every profile enables every method.

Tune result count, locale, and content length to the question. A retrieved page is evidence to assess: check the source and event date, retain citations, and acknowledge conflicting results.

**Exam cue:** search discovers a URL; Browse reads a known URL. Tool availability and fluent output do not prove a claim is correct.

[Back to module](README.md)
