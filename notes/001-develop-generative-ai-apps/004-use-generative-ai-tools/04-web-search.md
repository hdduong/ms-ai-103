# 04 Use the web_search tool

[Use the web_search tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/04-web-search?pivots=text)

- The model searches the public web when the answer depends on current facts. It writes the query, reads pages, and answers from those sources.
- In the playground this tool is **Grounding with Bing**. It can add cost, and customer data may leave the Azure compliance boundary.

![Playground tools pane with Web search attached, including the Bing grounding warning.](../003-foundry-sdk/images/02-playground.png)
- Say "latest" or a date range when freshness matters. Ask for official sources on high-stakes facts, then check them yourself.
- The same prompt can return a different answer later. Some regions or policies block web access. Search adds latency and tokens.

```python
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are an AI assistant. Use web search when current information is required.",
    input="What are three major announcements from Microsoft Build this week?",
    tools=[
        {"type": "web_search"},
    ],
)
print(response.output_text)
```
