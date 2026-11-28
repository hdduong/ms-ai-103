# 03 Translate text

[Translate text with Azure Translator](https://learn.microsoft.com/en-us/training/modules/translate-text-speech/3-azure-translator)

`TextTranslationClient` takes the resource endpoint, or a region plus a key. The lab uses Entra.

```python
from azure.ai.translation.text import TextTranslationClient
from azure.ai.translation.text.models import InputTextItem
from azure.identity import DefaultAzureCredential

client = TextTranslationClient(
    credential=DefaultAzureCredential(),
    endpoint="https://{resource}.cognitiveservices.azure.com/",
)
```

`get_supported_languages(scope="translation")` is the list of target codes. `translate` detects the source language unless you set it. One call can target several languages.

```python
results = client.translate(
    body=[InputTextItem(text="Hola")],
    to_language=["fr", "en"],
)
for item in results:
    source = item.detected_language.language
    for translated in item.translations:
        print(source, translated.to, translated.text)
```

**Transliteration** changes script, not language. Japanese `Jpan` to Latin `Latn`:

```python
client.transliterate(
    body=[InputTextItem(text="こんにちは")],
    language="ja",
    from_script="Jpan",
    to_script="Latn",
)
```

## Lab: pick a target, then loop

Lab: [Translate text and speech](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/07-translation.html). `translate-text.py` asks for a code such as `en`, rejects codes that are not in `languages.translation`, then translates until you type `quit`.
