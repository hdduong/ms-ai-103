# 04 Text to speech

[Text to speech](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/4-text-to-speech)

`AudioOutputConfig(use_default_speaker=True)` plays the audio. `filename=` writes a wav instead. Success is `SynthesizingAudioCompleted`. `Canceled` plus `cancellation_details` is the failure path.

```python
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
synthesizer = speech_sdk.SpeechSynthesizer(
    speech_config=speech_config,
    audio_config=audio_config,
)
result = synthesizer.speak_text_async("My voice is my password!").get()
if result.reason == speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print("Speech synthesized")
elif result.reason == speech_sdk.ResultReason.Canceled:
    print(result.cancellation_details.reason)
```

## Lab: save a greeting

From the [voice mail lab](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/04-azure-speech.html). Option 1 in the menu.

```python
output_file = "greeting.wav"
audio_config = speech_sdk.audio.AudioOutputConfig(filename=output_file)
speech_config.speech_synthesis_voice_name = "en-US-Serena:DragonHDLatestNeural"
synthesizer = speech_sdk.SpeechSynthesizer(
    speech_config=speech_config,
    audio_config=audio_config,
)
result = synthesizer.speak_text_async(greeting_message).get()
```

The greeting to try: `Hi. The person you called is not available right now. Leave a message.`
