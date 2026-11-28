# 02 Voice Live API

[Use the Voice Live API](https://learn.microsoft.com/en-us/training/modules/develop-voice-live-agent/2-voice-live-api)

Voice Live is a **WebSocket** for a live conversation: speech recognition, synthesis, avatar video, and audio processing in one session.

- Messages are JSON events. **Client events** go up. **Server events** come down.
- Audio formats include **PCM16** and **G.711**. Noise reduction and echo cancellation are built in.
- Voices can be OpenAI voices or Azure custom voices. An avatar uses **WebRTC**.
- Use a Foundry resource. The caller needs **Cognitive Services User**.
- Entra token scope is `https://ai.azure.com/.default`, or the older `https://cognitiveservices.azure.com/.default`. Send it as `Authorization: Bearer <token>`.
- A key works too, either as a header or a query value. Prefer the token.

Supported models and regions are in the [Voice Live overview](https://learn.microsoft.com/azure/ai-services/speech-service/voice-live).
