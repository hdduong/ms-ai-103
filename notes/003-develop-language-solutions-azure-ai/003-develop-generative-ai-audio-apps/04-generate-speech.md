# 04 Generate speech with a generative model

[Develop a speech app](https://learn.microsoft.com/en-us/training/modules/develop-generative-ai-audio-apps/3b-develop-speech-app)

Same `AzureOpenAI` client as [03](03-transcribe-speech.md). The deployment is `gpt-4o-mini-tts`. `voice` is an OpenAI voice name such as `alloy`, and `instructions` sets the tone.

## Lab: save a wav

Lab: [Generate and transcribe speech](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/03-gen-ai-speech.html).

```python
with client.audio.speech.with_streaming_response.create(
    model=model_deployment,  # gpt-4o-mini-tts deployment name
    voice="alloy",
    input="My voice is my passport!",
    instructions="Speak in a serious tone.",
) as response:
    # Write the audio bytes to disk as they arrive
    response.stream_to_file("output_speech.wav")
```

The Learn unit's sample uses `api_key=` and "Speak in an upbeat, excited tone." The lab uses Entra and a serious tone. Either client shape works if the deployment supports it.
