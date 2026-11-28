# 04 Translate speech

[Translate speech with Azure Speech](https://learn.microsoft.com/en-us/training/modules/translate-text-speech/4-speech-translation)

`SpeechTranslationConfig` is a `SpeechConfig` plus a source language and one or more targets. `add_target_language` uses a language code (`fr`), not a locale (`fr-FR`).

```python
import azure.cognitiveservices.speech as speech_sdk
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
translation_cfg = speech_sdk.translation.SpeechTranslationConfig(
    token_credential=credential,
    endpoint="https://{resource}.cognitiveservices.azure.com/",
)
translation_cfg.speech_recognition_language = "en-US"  # what the user speaks
translation_cfg.add_target_language("fr")
translation_cfg.add_target_language("ja")

audio_cfg = speech_sdk.AudioConfig(use_default_microphone=True)
translator = speech_sdk.translation.TranslationRecognizer(
    translation_config=translation_cfg,
    audio_config=audio_cfg,
)
result = translator.recognize_once_async().get()
# result.text is the source transcript. translations maps each target code to text
for code, text in result.translations.items():
    print(code, text)
```

To speak a translation, create a separate `SpeechSynthesizer` and set `speech_synthesis_voice_name` per language (`fr-FR-HenriNeural`, `ja-JP-NanamiNeural`).

Setting `translation_cfg.voice_name` and handling the `synthesizing` event writes the spoken translation to a wav in one step, without a second synthesizer.

## Lab: French, Spanish, and Hindi

From the [translation lab](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/07-translation.html). `translate-speech.py` recognizes US English and prints plus speaks three targets:

```python
voices = {
    "fr": "fr-FR-HenriNeural",
    "es": "es-ES-ElviraNeural",
    "hi": "hi-IN-MadhurNeural",
}
speech_cfg = speech_sdk.SpeechConfig(
    token_credential=credential,
    endpoint=foundry_endpoint,
)
for language, text in result.translations.items():
    speech_cfg.speech_synthesis_voice_name = voices[language]
    synthesizer = speech_sdk.SpeechSynthesizer(speech_cfg, audio_out_cfg)
    synthesizer.speak_text_async(text).get()
```
