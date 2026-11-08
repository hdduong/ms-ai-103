# Microsoft Learn review

**Reviewed: 2026-11-07 (America/Los_Angeles).**

This update reviews the two learning paths already represented in this repository and adds their exercise material. It covers the six generative AI modules, the nine existing agents modules, and the newly added Web IQ module.

## Sources and freshness

| Source | Version checked | Purpose |
| --- | --- | --- |
| [Generative AI path](https://learn.microsoft.com/en-us/training/paths/develop-generative-ai-apps/) | Live page on the review date; 6 modules | Module and lesson alignment |
| [Agents path](https://learn.microsoft.com/en-us/training/paths/develop-ai-agents-azure/) | Live page on the review date; 10 modules | Module and lesson alignment |
| [MicrosoftLearning/mslearn-ai-studio](https://github.com/MicrosoftLearning/mslearn-ai-studio/tree/b702dc2fd9f9411b5eb1f86b9259db6515cd9c48) | `b702dc2fd9f9411b5eb1f86b9259db6515cd9c48`, commit dated 2026-08-19 | Generative AI exercise instructions and starter files |
| [MicrosoftLearning/mslearn-ai-agents](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c) | `4d6a32629722927bd5f1034e0b5396e0da4e962c`, commit dated 2026-11-04 | Agent exercise instructions, package requirements, and starter files |
| [AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103) | Live page last updated 2026-10-31; skills heading dated 2026-04-16 | Exam coverage check |

Each exercise lesson links to both the live exercise and a pinned source snapshot. The snapshot records what was reviewed; it is not a claim that those versions will remain the latest. Copied/adapted code is attributed in [third-party notices](../THIRD_PARTY_NOTICES.md).

## Changes

- Added the missing [Web IQ module](002-develop-ai-agents-azure/010-get-started-web-iq/README.md) at its current place in the agents path. Existing folder names remain stable to preserve links.
- Added [exercise lessons and a code index](EXERCISES.md), with code explanations, package/setup requirements, and source references. Portal exercises are identified explicitly.
- Corrected the Foundry SDK example to `get_openai_client()` and explained the difference from the dated API-version argument still shown in its training unit. Added complete Responses, Chat Completions, streaming, async, web/file-search, and local-function examples.
- Corrected deployment descriptions using the current product reference, including Azure geography processing, the Batch completion target, and the temporary Developer tier.
- Kept exercise requirements separate. A Foundry project client, a direct model client, and an Agent Framework client are different entry points; examples must use their matching endpoint and credentials.
- Reconciled Agent Framework examples with the pinned lab packages: `FoundryChatClient`, `as_agent`, current orchestration builders, and workflow event handling. The exercise notes identify differences from older conceptual examples.
- Made environment-variable names explicit where adjacent labs use different conventions or lab prose and checked-in code disagree.
- Fixed the local MCP bridge for SDK field-name changes by reading serialized protocol aliases (`model_dump(by_alias=True)`). Local stdio execution verified discovery and both tools against the installed MCP/FastMCP versions.
- Updated the Work IQ lesson to the current remote MCP/OAuth exercise, with its optional lab separate from Teams/Copilot publishing.
- Retained the currently published workflow lab with an explicit migration notice: Microsoft documents retirement of the visual designer and in-portal workflow execution after **2026-12-01**. Exported YAML can run as a hosted agent; Microsoft Agent Framework and Azure Logic Apps are documented migration paths. See [the official migration guide](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/workflow#migration-guide).
- Included source snapshots and license notices so future updates can be compared with a known baseline.

## Package and source cautions

The numbered agent labs in the reviewed snapshot include an `openai<3` constraint where they use the OpenAI client with Azure AI Projects. The repository comments explain an HTTP client compatibility issue. Use the requirements file linked from the relevant exercise rather than replacing it with an unrestricted upgrade.

The Agent Framework labs use their own versioned `agent-framework-*` packages. The repository's `semantic-kernel` folder names follow historical Microsoft Learn URL slugs; they do not mean the current lessons should install Semantic Kernel.

For source details, see the [agent lab requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/02-agent-custom-tools/Python/requirements.txt), [single-agent requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/07-agent-framework/python/requirements.txt), and [orchestration requirements](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/08-agent-orchestration/Python/requirements.txt).

## Remaining AI-103 coverage

This table is a study-planning assessment of the `notes/` lessons, not a claim of full exam coverage. Third-party practice questions elsewhere in the repository are not a substitute for completing the official learning material.

| Exam area | Current weight | What these notes cover; what still needs dedicated study |
| --- | --- | --- |
| Planning and management | 25–30% | Foundry setup, models, deployments, evaluation, and responsible AI. Deepen CI/CD, quotas/cost, networking, identity, and operational monitoring. |
| Generative AI and agents | 30–35% | The main focus of the two paths. Practice deployment, tool use, orchestration, grounding, evaluation, and error handling. |
| Computer vision | 10–15% | No dedicated notes path yet. Add image/video generation and editing, multimodal analysis, and visual safety. |
| Text analysis | 10–15% | No dedicated notes path yet. Add language analysis, translation, speech, and audio workflows. |
| Information extraction | 10–15% | Foundry IQ/RAG provide grounding context. Add indexing/enrichment, OCR/layout, document extraction, and Content Understanding analyzers. |

Weights and scope come from the [current AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103).

## Validation and practical limits

Final checks covered **113 Markdown files** (the notes plus the root README and notices):

- **67 Python code blocks** passed syntax parsing; **23 Bash blocks**, **6 JSON blocks**, and **1 JSONL block** passed their format checks.
- **268 local Markdown links** resolve. **127 distinct Microsoft Learn, Microsoft Learning exercise-site, and package-reference URLs** returned HTTP 200.
- **46 distinct pinned upstream paths** exist in the inspected Microsoft Learning source snapshots. `git diff --check` passes.
- The astronomy functions passed price calculations, invalid-input handling, report creation, and filename checks.
- An actual local MCP stdio session passed initialization, discovery, both tool calls, schema preservation, and the expected inventory classifications. Completed core client programs imported, and Azure tool/agent definitions serialized without making service calls.
- Tool-loop tests verified that completion after the eighth round is accepted before a ninth round is rejected.

The local core-example environment used Python 3.12 with `azure-ai-projects==2.3.0`, `openai==2.54.0`, `mcp==2.3.0`, and `fastmcp==4.1.6`. The lessons retain the official exercises' Python 3.13 setup. Framework API checks used the pinned package source; a dependency-resolution dry run also succeeded for the multi-agent lab.

Azure-dependent examples require the learner's deployed resources, model quota, role assignments, and credentials. They have not been executed against an Azure subscription as part of this review. No tenant configuration, Microsoft 365 publication, model inference, paid evaluation, or fine-tuning was performed.

An excerpt can require surrounding lab setup even when its Python syntax is valid. Exercise lessons identify their setup and distinguish complete examples, excerpts, and portal-only work.

[Back to notes](README.md)
