# 02 Understand the Speech MCP server

[Understand the Azure Speech MCP server](https://learn.microsoft.com/en-us/training/modules/develop-speech-agent-speech-mcp/02-understand-speech-mcp)

Two tools. The agent picks from the descriptions. You do not route.

| Tool | Input | Output |
| --- | --- | --- |
| Speech-to-text (Recognize) | WAV, MP3, OGG, FLAC, MP4, M4A, AAC, and similar | Transcript. Options include language, phrase hints, and profanity filtering |
| Text-to-speech (Synthesize) | Text, plus a voice such as `en-US-JennyNeural` | An audio file, WAV or MP3 |

Audio does not travel inside the chat message. You need a blob container:

- Text-to-speech writes the file there and the agent returns a link.
- Speech-to-text reads a public URL or a blob SAS URL.

The connection stores a **SAS URL** with read, write, add, create, and list. Treat it as a secret: shortest expiry, one container, never in instructions or source. The connection is key-based. You also need **Storage Blob Data Contributor** on the account. If the tool is missing from the catalog, the Foundry resource is in a region that does not offer it.

```text
https://{foundry-resource-name}.cognitiveservices.azure.com/speech/mcp
```
