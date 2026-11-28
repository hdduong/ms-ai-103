# 04 Extract entities

[Extract entities](https://learn.microsoft.com/en-us/training/modules/analyze-text-ai-language/6-extract-entities)

Named entity recognition finds mentions and labels them. Categories to remember: **Person**, **PersonType**, **Location**, **DateTime**, **Organization**, **Address**, **Email**, **URL**. The [category list](https://learn.microsoft.com/azure/ai-services/language-service/named-entity-recognition/concepts/named-entity-categories) is longer.

```python
documents = [
    "Microsoft was founded on April 4, 1975 by Bill Gates and Paul Allen in Albuquerque, New Mexico.",
    "Satya Nadella became CEO of Microsoft on February 4, 2014.",
]
response = client.recognize_entities(documents=documents)
for doc in response:
    for entity in doc.entities:
        # text is the mention; category is the label
        print(entity.text, entity.category)
```

From those sentences: Microsoft is Organization, April 4, 1975 is DateTime, Bill Gates and Paul Allen are Person, Albuquerque and New Mexico are Location, CEO is PersonType.
