# 03 Detect language

[Detect language](https://learn.microsoft.com/en-us/training/modules/analyze-text-ai-language/3-detect-language)

- Each document comes back with a primary language, an ISO 639-1 code, and a confidence from 0 to 1.
- Use it when the language is unknown: a content store, or the first turn of a chat so later replies match the user.
- Limits: under **5,120 characters** per document, and at most **1,000** documents in one call.
- Mixed-language text returns the language that occupies the most of the document, with a **lower** confidence.
- Text the service cannot parse comes back as `(unknown)` with score `0`.

```python
documents = ["Hello World!", "Bonjour le monde!"]
# One result per document, in the same order
response = client.detect_language(documents=documents)
for doc in response:
    lang = doc.primary_language
    print(doc.id, lang.name, lang.iso6391_name, lang.confidence_score)
```

`Hello World!` is English `en` at about 0.9. `Bonjour le monde!` is French `fr` at about 0.98.
