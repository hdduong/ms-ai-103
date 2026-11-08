# 06 Exercise: Create a generative AI chat app

[Microsoft Learn exercise](https://learn.microsoft.com/en-us/training/modules/foundry-sdk/06-exercise)

> Reviewed **2026-11-07**. The official lab builds Chat Completions and Responses API clients, then adds conversation state, streaming, and async calls. Allow about **45 minutes**.

## 1. Prepare the deployment and environment

Create a project in **New Foundry**, deploy **gpt-5.2**, and record its deployment name and **Azure OpenAI endpoint**. This exercise uses the `openai` package directly. Its endpoint is the full OpenAI v1 base URL, **not the Foundry project URL**.

The official prerequisites are Azure CLI, Git, VS Code, and Python **3.13.x** (tested **3.13.12**). To reproduce the reviewed source on macOS/Linux:

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-studio.git
cd mslearn-ai-studio
git checkout b702dc2fd9f9411b5eb1f86b9259db6515cd9c48
cd labfiles/foundry-chat/python/chat-app
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
az login
```

On Windows, create the environment with `py -3.13 -m venv .venv` and activate `.venv\Scripts\Activate.ps1` in PowerShell. If required, use `az login --tenant <tenant-id>` for the resource's tenant. Your signed-in account needs access to the deployed model.

The pinned lab's `requirements.txt` contains exactly:

```text
python-dotenv
aiohttp
azure-identity
openai
```

These names are **not a version lock**. Use a separate virtual environment for this lab; do not assume its unpinned dependencies are interchangeable with another lab's `azure-ai-projects` environment. This review checked source and syntax, not compatibility of every newly released package version.

Create `.env` beside the Python files:

```dotenv
AZURE_OPENAI_ENDPOINT="https://<resource-name>.openai.azure.com/openai/v1/"
MODEL_DEPLOYMENT="<your-gpt-5.2-deployment-name>"
```

The endpoint and deployment are configuration, not credentials. `DefaultAzureCredential` obtains a Microsoft Entra token after Azure CLI sign-in. The lab uses scope `https://ai.azure.com/.default`.

## 2. Complete Responses API chat application

The upstream `chat-app.py` is a starter with TODOs. This assembled version combines the official exercise's code blocks and closes the client and credential with context managers.

```python
import os

from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import OpenAI


def main():
    load_dotenv()
    endpoint = os.environ["AZURE_OPENAI_ENDPOINT"]
    deployment = os.environ["MODEL_DEPLOYMENT"]
    instructions = "You are a helpful AI assistant."
    last_response_id = None

    with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        with OpenAI(base_url=endpoint, api_key=token_provider) as client:
            while True:
                prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue

                response = client.responses.create(
                    model=deployment,
                    instructions=instructions,
                    input=prompt,
                    previous_response_id=last_response_id,
                )
                print(response.output_text)
                last_response_id = response.id


if __name__ == "__main__":
    main()
```

Save it as `chat-app.py` in the lab directory, then run:

```bash
python chat-app.py
```

Ask about **ELIZA**, then ask `How does it compare to modern LLMs?` With `previous_response_id`, the second turn has the earlier response as context. Remove that parameter for the lab's stateless comparison.

**What matters:** `model` is the deployment name; `input` is this turn's message; `output_text` combines textual response content; and `response.id` connects later turns. Send `instructions` on every call: prior instructions are not automatically carried forward by `previous_response_id`.

## 3. Compare Chat Completions

Inside the same configured client's loop, replace the Responses API call and print with this block to reproduce the lab's stateless Chat Completions step:

```python
completion = client.chat.completions.create(
    model=deployment,
    messages=[
        {"role": "system", "content": instructions},
        {"role": "user", "content": prompt},
    ],
)
print(completion.choices[0].message.content)
```

This two-message array contains no earlier turns. For a multi-turn Chat Completions application, maintain and send the conversation's message history yourself. Do not pass Responses API fields such as `previous_response_id` to Chat Completions.

## 4. Stream the Responses API output

Alternatively, replace the non-streaming Responses API call and its two following lines in the complete app with:

```python
stream = client.responses.create(
    model=deployment,
    instructions=instructions,
    input=prompt,
    previous_response_id=last_response_id,
    stream=True,
)
for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
    elif event.type == "response.completed":
        last_response_id = event.response.id
print()
```

Text delta events let the UI display output as it arrives. Save the completed response's ID for the next turn. A production app also needs to surface interrupted/failed streams instead of treating partial text as a completed response.

## 5. Async variant

The lab also provides `chat-async.py` as a starter. Use the **async identity namespace** with `AsyncOpenAI`; do not mix a synchronous credential provider into this example. This assembled variant uses the same `.env` and requirements:

```python
import asyncio
import os

from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
from dotenv import load_dotenv
from openai import AsyncOpenAI


async def main():
    load_dotenv()
    endpoint = os.environ["AZURE_OPENAI_ENDPOINT"]
    deployment = os.environ["MODEL_DEPLOYMENT"]
    last_response_id = None

    async with DefaultAzureCredential() as credential:
        token_provider = get_bearer_token_provider(
            credential, "https://ai.azure.com/.default"
        )
        async with AsyncOpenAI(base_url=endpoint, api_key=token_provider) as client:
            while True:
                prompt = input("Enter a prompt (or 'quit' to exit): ").strip()
                if prompt.lower() == "quit":
                    break
                if not prompt:
                    continue

                response = await client.responses.create(
                    model=deployment,
                    instructions="You are a helpful AI assistant.",
                    input=prompt,
                    previous_response_id=last_response_id,
                )
                print(response.output_text)
                last_response_id = response.id


if __name__ == "__main__":
    asyncio.run(main())
```

Save as `chat-async.py` and run `python chat-async.py`. This console demo still waits for keyboard input; the async API call is the part useful when integrating into a concurrent application.

## Checks and cleanup

- Verify the endpoint ends in `/openai/v1/` and `MODEL_DEPLOYMENT` matches a deployed model.
- Test a follow-up question to check conversation continuity, and `quit` to exit.
- Do not add sampling parameters indiscriminately: support differs between models, especially reasoning models. Start with the lab's supported request shape.
- Delete only the disposable resource group created for the exercise when it is no longer needed.

## Source and attribution

Source: Microsoft's [Create a generative AI chat app](https://github.com/MicrosoftLearning/mslearn-ai-studio/blob/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/Instructions/Exercises/03-foundry-sdk.md), [starter files and requirements](https://github.com/MicrosoftLearning/mslearn-ai-studio/tree/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48/labfiles/foundry-chat/python/chat-app), commit `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`. Code is assembled/adapted from the exercise; changes add explicit configuration lookup, blank-input handling, and resource cleanup. See [third-party notices and MIT license](../../../THIRD_PARTY_NOTICES.md).

Validation during this review: Python syntax and source comparison. No authenticated Azure calls, model deployments, or paid lab runs were executed.
