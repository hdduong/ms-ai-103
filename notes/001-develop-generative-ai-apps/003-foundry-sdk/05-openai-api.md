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
    # Read MODEL_DEPLOYMENT and AZURE_OPENAI_ENDPOINT from .env
    load_dotenv()
    deployment = os.environ["MODEL_DEPLOYMENT"]
    # Sent as the system message on every request
    instructions = "You are a helpful AI assistant."

    # Same Entra ID sign-in and client as the Responses program
    with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        with OpenAI(
            base_url=os.environ["AZURE_OPENAI_ENDPOINT"],
            api_key=token_provider,
        ) as client:
            # One question per pass until you type quit
            while True:
                prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue
                # Fresh list each turn: system rules and this question, no history
                completion = client.chat.completions.create(
                    model=deployment,  # deployment name
                    messages=[
                        {"role": "system", "content": instructions},
                        {"role": "user", "content": prompt},
                    ],
                )
                # The reply text is in the first choice's message
                print(completion.choices[0].message.content)


if __name__ == "__main__":
    main()
```

Ask about ELIZA, then ask a follow-up that refers to it. The second request sends no first-turn context, even though both answers appear in the same terminal.

If editing the Responses file instead, replace the call **and both following lines**, including `last_response_id = response.id`; remove its initialization too. Leaving that assignment behind refers to a `response` variable that the Chat Completions version never creates.

## Next turn: keep history in the application

To make the program conversational, replace the entire `while True` loop above with this block, at the same indentation inside `with OpenAI(...) as client:`. The `messages` list is initialized **once, before the loop**:

```python
# History lives in the app; create it once, before the loop
messages = [{"role": "system", "content": instructions}]
while True:
    prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
    if prompt.lower() == "quit":
        break
    if not prompt:
        continue
    # Add this question to the history before sending
    messages.append({"role": "user", "content": prompt})
    # Send the whole history so the model can resolve follow-ups
    completion = client.chat.completions.create(
        model=deployment,  # deployment name
        messages=messages,  # system, then every user/assistant turn so far
    )
    # content can be None, so fall back to an empty string
    assistant_text = completion.choices[0].message.content or ""
    print(assistant_text)
    # Keep the reply too, or the model can't refer to its own answer
    messages.append({"role": "assistant", "content": assistant_text})
```

| Conversation choice | What the next request sends |
| --- | --- |
| Lab's stateless Chat Completions | System instructions and this question |
| Chat Completions with `messages` history | System instructions and all retained user/assistant turns |
| Responses with `previous_response_id` | This question and a reference to server-side response context |

Both conversational approaches consume context tokens. For long conversations, decide what history to retain or summarize rather than growing the context indefinitely.
