# 05 Chat Completions API

[Generate responses with the Chat Completions API](https://learn.microsoft.com/en-us/training/modules/foundry-sdk/05-openai-api?pivots=text)

- Use this API for an existing Chat Completions integration or a compatible model that does not support Responses. Learn recommends Responses for new Foundry apps.
- The same OpenAI-compatible client can call either API. The request and response shapes differ.
- `messages` is a list: `system` supplies instructions, `user` supplies a question, and `assistant` supplies a previous reply.
- Read `completion.choices[0].message.content`. Chat Completions has no `output_text` or `previous_response_id`.

The playground's **Continue in code** action can generate either API ([unit 02](02-chat-playground.md)).

## Lab: swap in Chat Completions

The [Create a generative AI chat app](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/03-foundry-sdk.html) exercise starts with a stateless Chat Completions call. Compare it with the [Responses program](04-responses-api.md#lab-chat-app): authentication, endpoint, and deployment stay the same; the two-message array replaces `input` and `instructions`.

Use that lesson's environment setup and `.env`. Save the following complete alternative as `chat-completions.py` and run `python chat-completions.py`:

```python
import os

from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import OpenAI


def main():
    load_dotenv()
    deployment = os.environ["MODEL_DEPLOYMENT"]
    instructions = "You are a helpful AI assistant."

    with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        with OpenAI(
            base_url=os.environ["AZURE_OPENAI_ENDPOINT"],
            api_key=token_provider,
        ) as client:
            while True:
                prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue
                completion = client.chat.completions.create(
                    model=deployment,  # deployment name
                    messages=[
                        {"role": "system", "content": instructions},
                        {"role": "user", "content": prompt},
                    ],
                )
                print(completion.choices[0].message.content)


if __name__ == "__main__":
    main()
```

Each iteration constructs a fresh list containing only the system message and the current question. Ask about ELIZA, then ask a follow-up that refers to it. The second request sends no first-turn context, even though both answers appear in the same terminal.

If editing the Responses file instead, replace the call **and both following lines**, including `last_response_id = response.id`; remove its initialization too. Leaving that assignment behind refers to a `response` variable that the Chat Completions version never creates.

## Next turn: keep history in the application

To make the program conversational, replace the entire `while True` loop above with this block, at the same indentation inside `with OpenAI(...) as client:`. The `messages` list is initialized **once, before the loop**:

```python
messages = [{"role": "system", "content": instructions}]
while True:
    prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
    if prompt.lower() == "quit":
        break
    if not prompt:
        continue
    messages.append({"role": "user", "content": prompt})
    completion = client.chat.completions.create(
        model=deployment,  # deployment name
        messages=messages,
    )
    assistant_text = completion.choices[0].message.content or ""
    print(assistant_text)
    messages.append({"role": "assistant", "content": assistant_text})
```

After the first reply, the list holds `system → user → assistant`. After the next question, it holds `system → user → assistant → user`, so the model receives the context needed to resolve a follow-up. Append the assistant reply as well as the user question; otherwise the model cannot refer to its own earlier answer.

| Conversation choice | What the next request sends |
| --- | --- |
| Lab's stateless Chat Completions | System instructions and this question |
| Chat Completions with `messages` history | System instructions and all retained user/assistant turns |
| Responses with `previous_response_id` | This question and a reference to server-side response context |

Both conversational approaches consume context tokens. For long conversations, decide what history to retain or summarize rather than growing the context indefinitely.
