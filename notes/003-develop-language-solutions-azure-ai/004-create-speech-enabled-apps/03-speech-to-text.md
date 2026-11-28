# 03 Speech to text

[Speech to text](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/3-speech-to-text)

`AudioConfig` is the source. Omit it and the SDK uses the microphone. `recognize_once_async().get()` waits for one utterance.

```python
audio_config = speech_sdk.audio.AudioConfig(filename="audio.wav")
recognizer = speech_sdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config,
)
result = recognizer.recognize_once_async().get()
# RecognizedSpeech means result.text is the transcript
if result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    print(result.text)
else:
    print(result.reason)
```

## Lab: transcribe the messages folder

Lab: [Create a voice mail app](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/04-azure-speech.html). The greeting is written in [04](04-text-to-speech.md#lab-save-a-greeting). This function reads each file in `messages`.

```python
audio_config = speech_sdk.audio.AudioConfig(filename=file_path)
speech_recognizer = speech_sdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config,
)
result = speech_recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    print("Transcription:", result.text)
```
