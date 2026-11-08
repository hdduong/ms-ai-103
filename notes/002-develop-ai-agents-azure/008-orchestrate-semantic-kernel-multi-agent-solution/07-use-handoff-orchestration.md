# 07 Use handoff orchestration

[Use handoff orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/7-use-handoff-orchestration?pivots=text)

- One agent works, then passes the conversation to a specialist. The next agent is not known at the start.
- Use it when the domain shows up in the text (billing versus technical) and you can name the handoff rule.
- Skip it when the order is fixed, a simple rule is enough, a wrong route annoys the user, work must run in parallel, or agents can bounce forever.
- The Learn unit illustrates deterministic routing with a classifier and switch-case edge; include a default case. The SDK also offers `HandoffBuilder`, where agents can request a transfer through generated handoff tools. These are two ways to design routing, not the same API.

## Classifier, then a switch-case edge

This is the unit's classifier-and-routing design in code. The billing and technical steps below return labels; they do not contact real support teams. The classifier returns typed JSON, a small executor turns it into a route, and `add_switch_case_edge_group` checks the cases in order. `Default` catches anything that does not match. Pattern from the [switch-case control-flow sample](https://github.com/microsoft/agent-framework/tree/main/python/samples/03-workflows/control-flow).

Use the pinned packages and `.env` from [09-exercise.md](09-exercise.md). This is a supplementary pattern example; the official lab itself implements the sequential feedback pipeline.

```python
import asyncio
import os
from dataclasses import dataclass
from typing import Any, Literal, Never

from agent_framework import (
    Agent,
    AgentExecutor,
    AgentExecutorRequest,
    AgentExecutorResponse,
    Case,
    Default,
    Message,
    WorkflowBuilder,
    WorkflowContext,
    executor,
)
from agent_framework.foundry import FoundryChatClient
from agent_framework.openai import OpenAIChatOptions
from azure.identity import AzureCliCredential
from dotenv import load_dotenv
from pydantic import BaseModel

class Route(BaseModel):
    kind: Literal["billing", "technical", "unknown"]

@dataclass
class Routed:
    kind: str
    text: str

def is_kind(kind: str):
    def condition(message: Any) -> bool:
        return isinstance(message, Routed) and message.kind == kind
    return condition

@executor(id="start")
async def start(ticket: str, ctx: WorkflowContext[AgentExecutorRequest]) -> None:
    ctx.set_state("ticket", ticket)
    await ctx.send_message(
        AgentExecutorRequest(messages=[Message("user", contents=[ticket])], should_respond=True)
    )

@executor(id="to_route")
async def to_route(response: AgentExecutorResponse, ctx: WorkflowContext[Routed]) -> None:
    route = Route.model_validate_json(response.agent_response.text)
    await ctx.send_message(Routed(kind=route.kind, text=ctx.get_state("ticket")))

@executor(id="billing")
async def billing(item: Routed, ctx: WorkflowContext[Never, str]) -> None:
    await ctx.yield_output(f"Billing team: {item.text}")

@executor(id="technical")
async def technical(item: Routed, ctx: WorkflowContext[Never, str]) -> None:
    await ctx.yield_output(f"Technical team: {item.text}")

@executor(id="fallback")
async def fallback(item: Routed, ctx: WorkflowContext[Never, str]) -> None:
    await ctx.yield_output(f"Ask the user to rephrase: {item.text}")

async def main():
    load_dotenv()
    classifier = AgentExecutor(
        Agent(
            client=FoundryChatClient(
                project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
                model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],  # deployment name
                credential=AzureCliCredential(),
            ),
            name="classifier",
            instructions="Return JSON with kind set to billing, technical, or unknown.",
            default_options=OpenAIChatOptions[Any](response_format=Route),
        )
    )
    workflow = (
        WorkflowBuilder(start_executor=start)
        .add_edge(start, classifier)
        .add_edge(classifier, to_route)
        .add_switch_case_edge_group(
            to_route,
            [
                Case(condition=is_kind("billing"), target=billing),
                Case(condition=is_kind("technical"), target=technical),
                Default(target=fallback),
            ],
        )
        .build()
    )
    result = await workflow.run("I was charged twice for the same invoice.")
    print(result.get_outputs())

asyncio.run(main())
```

The SDK also has `HandoffBuilder(participants=[...]).with_start_agent(triage).build()`. It gives each agent a handoff tool, so the agents pick the next specialist themselves instead of a fixed switch-case.
