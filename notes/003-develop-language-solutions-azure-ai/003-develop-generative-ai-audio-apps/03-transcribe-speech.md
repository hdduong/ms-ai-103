# 03 Transcribe speech with a generative model

[Develop an audio chat app](https://learn.microsoft.com/en-us/training/modules/develop-generative-ai-audio-apps/3-develop-audio-chat-app)

This is not the Speech SDK. You upload a file to a **gpt-4o transcribe** deployment through `AzureOpenAI`.

```python
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

# Entra token; the lab does not put a key in the client
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), "https://ai.azure.com/.default"
)
client = AzureOpenAI(
    azure_endpoint=endpoint,  # the model deployment's endpoint
    azure_ad_token_provider=token_provider,
    api_version="2025-03-01-preview",
)
audio_file = open("speech.mp3", "rb")
# response_format="text" returns the transcript string, not a JSON object
transcription = client.audio.transcriptions.create(
    model=model_deployment,  # gpt-4o-mini-transcribe deployment name
    file=audio_file,
    response_format="text",
)
print(transcription)
```

`.env` in the lab uses `MODEL_ENDPOINT` and `MODEL_NAME`.
