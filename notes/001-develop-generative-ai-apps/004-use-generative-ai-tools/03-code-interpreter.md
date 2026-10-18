# 03 Use the code_interpreter tool

[Use the code_interpreter tool](https://learn.microsoft.com/en-us/training/modules/use-generative-ai-tools/03-code-interpreter?pivots=text)

- Gives the model a sandbox where it writes and runs Python. It sees the result and can retry after an error.
- Good for math, CSV or JSON work, and format conversion. pandas, numpy, matplotlib, and math are available.
- No network from the sandbox. Long jobs can time out. Huge data may not fit in memory.
- In instructions, many models look for the name **python tool**.

```python
response = openai_client.responses.create(
    model="gpt-4.1",  # deployment name
    instructions="You are an AI assistant. Use the python tool to run code for math problems.",
    input="What is the square root of 16?",
    tools=[
        {"type": "code_interpreter", "container": {"type": "auto"}},
    ],
)
print(response.output_text)
```
