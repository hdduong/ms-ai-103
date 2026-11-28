# 05 Audio format and voices

[Configure audio format and voices](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/5-audio-format-voices)

Both settings live on `SpeechConfig`, before you create the synthesizer.

Format is file type, sample rate, and bit depth:

```python
speech_config.set_speech_synthesis_output_format(
    speech_sdk.SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm
)
```

A voice name is locale, person, and model. Example: `en-US-Brian:DragonHDLatestNeural`. The lab greeting uses `en-US-Serena:DragonHDLatestNeural`.

```python
speech_config.speech_synthesis_voice_name = "en-US-Brian:DragonHDLatestNeural"
```
