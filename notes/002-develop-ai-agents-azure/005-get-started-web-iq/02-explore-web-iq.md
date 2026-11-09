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

## Choose a method for the music-agent example

The [official exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/04b-web-iq.html) builds `band-agent`. A question about a band's history may be answerable from model knowledge. Asking about its next concert needs current evidence, so Web Search can find an official tour page and Browse can inspect that page. A request for pictures calls for Images; a text-only web result is not the same output.

The playground lets you observe this distinction before involving the model. Run the same artist query with **Web** and **Images** selected and compare the returned links and result types. Use query/site restrictions to focus on relevant sources, language/region to match the audience, and result count/content length to control how much evidence reaches the model. Content format controls whether to return focused passages or fuller documents.

Continue with [resource and credential setup](03-configure-web-iq.md#lab-create-the-resource), then [attach the tool and inspect the agent's choices](04-use-web-iq-foundry.md#lab-band-agent). The exercise is a portal workflow; its instructions and queries are embedded in those lessons.
