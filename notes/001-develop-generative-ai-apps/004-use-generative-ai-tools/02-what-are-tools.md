# 02 What are tools?

[What are tools?](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/02-what-are-tools?pivots=text)

- Deploy a model that supports **tool calling**, then pass a `tools` list on `responses.create`.
- By default the model decides whether to call a tool and which one. Instructions and tool-choice rules can steer that.
- Four tools in this module: `code_interpreter`, `web_search`, `file_search`, `function`.

```python
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are a helpful AI assistant.",
    input="Find me some information about vintage computers.",
    tools=[
        {"type": "web_search"},
    ],
)
print(response.output_text)
```
