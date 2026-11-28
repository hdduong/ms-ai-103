# 02 Deploy a speech-capable model

[Deploy a speech-capable generative AI model](https://learn.microsoft.com/en-us/training/modules/develop-generative-ai-audio-apps/2-deploy-multimodal-model)

Two jobs, two deployments from the model catalog:

| Job | Models in this module |
| --- | --- |
| Transcribe audio to text | `gpt-4o-transcribe`, `gpt-4o-mini-transcribe`, `gpt-4o-transcribe-diarize` |
| Turn text into speech | `gpt-4o-mini-tts` in the lab |

Availability depends on region, version, and quota. The lab deploys both in **East US 2**. `model` in code is the deployment name.
