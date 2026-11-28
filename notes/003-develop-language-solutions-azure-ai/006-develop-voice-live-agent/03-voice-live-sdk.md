# 03 Voice Live SDK

[Use the Voice Live SDK](https://learn.microsoft.com/en-us/training/modules/develop-voice-live-agent/3-voice-live-sdk)

`azure.ai.voicelive.connect` opens the WebSocket and closes it when the block ends.

```python
import asyncio
from azure.ai.voicelive import connect
from azure.identity.aio import DefaultAzureCredential

async def main():
    async with connect(
        endpoint="https://{resource}.cognitiveservices.azure.com/",
        credential=DefaultAzureCredential(),
        model="gpt-4o",  # deployment the session talks to
    ) as connection:
        async for event in connection:
            # ServerEventType.SESSION_UPDATED means the session can take audio
            print(event.type)

asyncio.run(main())
```

Events you will see in a real session:

| Event | Meaning |
| --- | --- |
| Session updated | The session is configured. Start the microphone |
| Input audio buffer speech started | The user began talking. Stop playback so they can interrupt |
| Response audio delta | A chunk of the agent's voice. Queue it for the speaker |

The microphone callback base64-encodes PCM and calls `connection.input_audio_buffer.append`. Clearing the playback queue is how the app handles an interruption.
