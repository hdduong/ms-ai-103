# 04 Use concurrent orchestration

[Use concurrent orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/4-use-concurrent-orchestration?pivots=text)

- Every agent gets the same task and works alone. Combine the replies, or leave them as separate side effects.
- Use it for brainstorming, ensemble reasoning, a vote, or when waiting in series is too slow.
- Skip it when the next step needs the previous answer, the order must be stable, quota cannot cover parallel calls, agents would write the same data at once, or you cannot merge conflicts.
- `ConcurrentBuilder(participants=[...]).build()`, then `run`, then `get_outputs()`. Each message has an author.

## Three reviewers, one task

`ConcurrentBuilder` is in `agent_framework.orchestrations`. The builder fans the prompt out to every participant and fans the replies back in. Pattern from the [Agent Framework orchestration samples](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/orchestrations).

Packages and `.env` as in the [lab](05-use-sequential-orchestration.md#lab-feedback-pipeline). The lab itself only builds the sequential pipeline.

```python
import asyncio
import os
from agent_framework import Agent, AgentResponse
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import ConcurrentBuilder
from azure.identity import AzureCliCredential
from dotenv import load_dotenv

async def main():
    load_dotenv()
    client = FoundryChatClient(
        project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],  # deployment name
        credential=AzureCliCredential(),
    )
    security = Agent(client=client, name="security", instructions="Review the design for security risks only.")
    cost = Agent(client=client, name="cost", instructions="Review the design for cost only.")
    reliability = Agent(client=client, name="reliability", instructions="Review the design for reliability only.")
    workflow = ConcurrentBuilder(participants=[security, cost, reliability]).build()
    result = await workflow.run("A web app calls one gpt-4.1 deployment in one region with an API key.")
    for output in result.get_outputs():
        if isinstance(output, AgentResponse):
            for message in output.messages:
                print(message.author_name, message.text)

if __name__ == "__main__":
    asyncio.run(main())
```


Save the program as `concurrent_review.py` and run it with the shared environment. The same design text reaches `security`, `cost`, and `reliability`; each instruction set constrains one review. The builder waits for the branches and exposes their results, while the author labels keep those results distinguishable.

Check that all three perspectives appear. Their completion order and wording may vary. No participant sees another participant's response in this pattern, so the program does not resolve disagreements between reviewers; a later aggregation step would do that.
