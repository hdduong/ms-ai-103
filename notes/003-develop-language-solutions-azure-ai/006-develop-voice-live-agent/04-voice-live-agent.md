# 04 Voice Live with a Foundry agent

[Use Voice Live with an agent](https://learn.microsoft.com/en-us/training/modules/develop-voice-live-agent/3b-voice-live-agent)

Connecting to a **model** means the session code owns the instructions. Connecting to an **agent** means the instructions, tools, and voice settings stay on the agent. The client only passes the agent id. That is the form to use when several voice experiences share one app.

In the agent playground, turn on **voice mode**. The configuration pane sets:

- language
- voice activity detection (when the user stopped, and when they interrupted)
- audio enhancement for noise
- voice, tone, and rate
- an interim spoken response while the model is still working
- an avatar

## Lab: connect the session to the agent

Lab: [Develop a Voice Live agent](https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/06-voice-live-agent.html). `.env` uses `AZURE_VOICELIVE_ENDPOINT`, `AZURE_VOICELIVE_PROJECT_NAME`, and `AZURE_VOICELIVE_AGENT_ID`. API version in the lab is `2026-01-01-preview`.

The starter builds `self.agent_config` from `.env` (`AZURE_VOICELIVE_ENDPOINT`, `AZURE_VOICELIVE_PROJECT_NAME`, `AZURE_VOICELIVE_AGENT_ID`). `start` then does five steps:

```python
async with connect(
    endpoint=self.endpoint,
    credential=self.credential,
    api_version="2026-01-01-preview",
    agent_config=self.agent_config,  # saved agent, not a bare model name
) as connection:
    self.connection = connection
    self.audio_processor = AudioProcessor(connection)  # mic and speaker
    await self.setup_session()  # formats, turn detection, noise reduction
    self.audio_processor.start_playback()
    await self.process_events()  # speech in, audio out, interruptions
```
