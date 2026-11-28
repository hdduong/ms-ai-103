# 05 Extract personal information

[Extract personal information](https://learn.microsoft.com/en-us/training/modules/analyze-text-ai-language/7-extract-personal-information)

- PII detection finds names, addresses, phones, emails, SSNs, credit cards, and also passwords, PINs, and ZIP codes.
- Each hit has a **category** and a **confidence_score**. You can set a threshold so only scores at or above it are returned.
- The same call returns `redacted_text`, with each PII span replaced by asterisks.

```python
documents = [
    "John Smith works at Contoso Ltd. His email is john.smith@contoso.com and his phone number is 555-012-456.",
]
response = client.recognize_pii_entities(documents=documents, language="en")
for doc in response:
    for entity in doc.entities:
        print(entity.text, entity.category, entity.confidence_score)
    # Same text, with the PII spans masked
    print(doc.redacted_text)
```

John Smith is Person, the email is Email, the phone is PhoneNumber. `123-45-6789` is USSocialSecurityNumber.

## Lab: review each file

Lab: [Analyze text](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/01-analyze-text.html). `azure-ai-textanalytics==5.3.0`. The reviews sit in a `reviews` folder. `FOUNDRY_ENDPOINT` is the **resource** endpoint.

```python
import os
from azure.ai.textanalytics import TextAnalyticsClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()
client = TextAnalyticsClient(
    endpoint=os.environ["FOUNDRY_ENDPOINT"],  # resource endpoint, no /api/projects
    credential=DefaultAzureCredential(),
)

for file_name in os.listdir("reviews"):
    text = open(os.path.join("reviews", file_name), encoding="utf8").read()
    print(file_name)
    # One document per call; [0] is that document's result
    detected = client.detect_language(documents=[text])[0]
    print("Language:", detected.primary_language.name)
    entities = client.recognize_entities(documents=[text])[0].entities
    for entity in entities:
        print(entity.text, entity.category)
    pii = client.recognize_pii_entities(documents=[text])[0]
    for entity in pii.entities:
        print(entity.text, entity.category)
    print(pii.redacted_text)
```

Passing every review in one `documents=[...]` list is the batch form of the same three calls.
