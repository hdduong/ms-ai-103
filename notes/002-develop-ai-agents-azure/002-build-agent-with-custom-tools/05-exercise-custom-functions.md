# 05 Exercise — use custom functions in an AI agent

[Current Microsoft Learn exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html) · [Reviewed instructions](https://github.com/MicrosoftLearning/mslearn-ai-agents/blob/4d6a32629722927bd5f1034e0b5396e0da4e962c/Instructions/Exercises/02-agent-custom-tools.md) · [Starter code and data](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/02-agent-custom-tools/Python)

Reviewed **2026-11-07** against upstream commit `4d6a32629722927bd5f1034e0b5396e0da4e962c`.

## What this lab teaches

The astronomy agent chooses from three functions: find a visible event, calculate a telescope rental estimate, and write a report. **Your application executes each local function** and returns a `function_call_output` with the matching `call_id`. A function schema alone cannot execute Python.

This completed example combines the instruction-page additions with the starter files. It also handles repeated tool rounds, rejects unknown function names, keeps every turn on one conversation, and confines generated reports to a local `reports` directory. Those are adaptations to the exercise, not extra service features.

The event and rate files are **sample data**. Dates contain month and day only; this exercise searches the remaining part of the current year and does not wrap into the next year. Do not treat its results as a live astronomy forecast. A rental estimate or generated report does not reserve a telescope or charge money.

## 1. Set up the project and Python environment

Use the Foundry Toolkit for VS Code to create/select a project and deploy a supported model. The current lab uses a deployment named `gpt-5`; use your actual deployment name. Copy the **project endpoint**, which ends in `/api/projects/<project-name>`.

The upstream lab specifies Python **3.13** (tested with 3.13.12; Python 3.14 is not supported by the lab's dependency set). In a new working directory:

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/02-agent-custom-tools/Python
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

Create `.env` locally:

```dotenv
PROJECT_ENDPOINT=https://<foundry-resource>.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

The project authenticates with Microsoft Entra ID through `DefaultAzureCredential`. Your signed-in identity needs access to the project. Never commit `.env` values or credentials. Run from this lab directory so the sample `data` files are present.

## 2. Complete `functions.py`

Replace the starter's incomplete function file with this implementation. It retains the exercise's data format and calculation, uses paths relative to the script, and validates arguments in Python as well as in the model's tool schema.

```python
import json
import math
import re
from datetime import datetime
from pathlib import Path
from uuid import uuid4

BASE = Path(__file__).resolve().parent


def load_rates(filename):
    rates = {}
    for line in (BASE / "data" / filename).read_text().splitlines():
        key, value = line.split("|")
        rates[key] = float(value)
    return rates


TELESCOPE_RATES = load_rates("telescope_rates.txt")
PRIORITY_MULTIPLIERS = load_rates("priority_multipliers.txt")


def next_visible_event(location: str) -> str:
    today = int(datetime.now().strftime("%m%d"))
    normalized = location.strip().lower().replace(" ", "_")
    candidates = []
    for line in (BASE / "data/events.txt").read_text().splitlines():
        name, event_type, date, locations = line.split("|")
        month, day = map(int, date.split("-"))
        visible_from = locations.split(";")
        if normalized in visible_from and month * 100 + day >= today:
            candidates.append((month * 100 + day, {
                "event": name, "type": event_type, "date": date,
                "visible_from": sorted(visible_from),
            }))
    if candidates:
        return json.dumps(min(candidates, key=lambda item: item[0])[1])
    return json.dumps({"message": f"No remaining sample event for {location}."})


def calculate_observation_cost(telescope_tier: str, hours: float,
                               priority: str) -> str:
    tier = telescope_tier.strip().lower()
    priority = priority.strip().lower()
    if tier not in TELESCOPE_RATES:
        return json.dumps({"error": "Unknown telescope tier."})
    if priority not in PRIORITY_MULTIPLIERS:
        return json.dumps({"error": "Unknown priority."})
    if isinstance(hours, bool) or not math.isfinite(hours) or hours <= 0:
        return json.dumps({"error": "Hours must be a positive finite number."})
    rate = TELESCOPE_RATES[tier]
    multiplier = PRIORITY_MULTIPLIERS[priority]
    base = rate * hours
    return json.dumps({
        "telescope_tier": tier, "hours": hours, "hourly_rate": rate,
        "priority": priority, "priority_multiplier": multiplier,
        "base_cost": base, "total_cost": base * multiplier,
    })


def generate_observation_report(event_name: str, location: str,
                                telescope_tier: str, hours: float,
                                priority: str, observer_name: str) -> str:
    cost = json.loads(calculate_observation_cost(telescope_tier, hours, priority))
    if "error" in cost:
        return json.dumps(cost)
    next_event = json.loads(next_visible_event(location))
    reports = BASE / "reports"
    reports.mkdir(exist_ok=True)
    safe_name = re.sub(r"[^a-z0-9_-]+", "_", event_name.lower()).strip("_")[:60]
    target = reports / f"report_{safe_name or 'observation'}_{uuid4().hex[:8]}.txt"
    text = (
        "CONTOSO OBSERVATORIES — SAMPLE OBSERVATION REPORT\n"
        f"Created: {datetime.now().isoformat(timespec='minutes')}\n"
        f"Observer: {observer_name}\nEvent: {event_name}\nLocation: {location}\n"
        f"Next sample event: {json.dumps(next_event)}\n"
        f"Telescope: {cost['telescope_tier']}\nHours: {cost['hours']}\n"
        f"Hourly rate: ${cost['hourly_rate']:.2f}\n"
        f"Priority multiplier: {cost['priority_multiplier']}\n"
        f"Base cost: ${cost['base_cost']:.2f}\n"
        f"Estimated total: ${cost['total_cost']:.2f}\n"
        "No reservation or payment has been made.\n"
    )
    target.write_text(text, encoding="utf-8")
    return json.dumps({"status": "Report generated", "file": str(target)})
```

## 3. Complete `agent.py`

All properties are required and `additionalProperties` is false for these strict schemas. The helper constructs the same JSON-schema structure as the exercise; descriptions still tell the model when and how to use a tool.

```python
import json
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import FunctionTool, PromptAgentDefinition
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from functions import (
    next_visible_event, calculate_observation_cost, generate_observation_report,
)

FUNCTIONS = {
    "next_visible_event": next_visible_event,
    "calculate_observation_cost": calculate_observation_cost,
    "generate_observation_report": generate_observation_report,
}


def function_tool(name, description, properties):
    return FunctionTool(
        name=name, description=description, strict=True,
        parameters={"type": "object", "properties": properties,
                    "required": list(properties), "additionalProperties": False},
    )


LOCATION = {"type": "string", "description": "Continent, e.g. south_america."}
COST_PROPERTIES = {
    "telescope_tier": {"type": "string", "enum": ["standard", "advanced", "premium"]},
    "hours": {"type": "number", "description": "Positive number of rental hours."},
    "priority": {"type": "string", "enum": ["low", "normal", "high", "urgent"]},
}
TOOLS = [
    function_tool("next_visible_event", "Find the next event in the sample data.",
                  {"location": LOCATION}),
    function_tool("calculate_observation_cost", "Calculate a telescope rental estimate.",
                  COST_PROPERTIES),
    function_tool("generate_observation_report", "Write a local sample observation report.",
                  {"event_name": {"type": "string"}, "location": LOCATION,
                   **COST_PROPERTIES, "observer_name": {"type": "string"}}),
]


def finish_turn(client, conversation_id, reference, user_input):
    response = client.responses.create(
        conversation=conversation_id, input=user_input, extra_body=reference,
    )
    for round_number in range(9):
        if response.status == "failed":
            raise RuntimeError(str(response.error))
        calls = [item for item in response.output if item.type == "function_call"]
        if not calls:
            return response.output_text
        if round_number == 8:
            raise RuntimeError("Stopped after eight function-call rounds.")
        outputs = []  # Only outputs for this response; never resend old call IDs.
        for call in calls:
            function = FUNCTIONS.get(call.name)
            if function is None:
                result = json.dumps({"error": f"Unknown function: {call.name}"})
            else:
                try:
                    result = function(**json.loads(call.arguments))
                except (ValueError, TypeError) as exc:
                    result = json.dumps({"error": str(exc)})
            outputs.append({"type": "function_call_output", "call_id": call.call_id,
                            "output": result})
        response = client.responses.create(
            conversation=conversation_id, input=outputs, extra_body=reference,
        )
    raise RuntimeError("Stopped after eight function-call rounds.")


def main():
    load_dotenv()
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=os.environ["PROJECT_ENDPOINT"],
                        credential=credential) as project,
        project.get_openai_client() as client,
    ):
        agent = project.agents.create_version(
            agent_name="astronomy-agent",
            definition=PromptAgentDefinition(
                model=os.environ["MODEL_DEPLOYMENT_NAME"],
                instructions=("You are an astronomy observations assistant. Use the tools "
                              "for sample events and rental estimates. Ask for missing "
                              "parameters. Generate a report only when asked. Explain "
                              "that the data are illustrative and no booking is made."),
                tools=TOOLS,
            ),
        )
        conversation = None
        try:
            conversation = client.conversations.create()
            reference = {"agent_reference": {"name": agent.name, "version": agent.version,
                                              "type": "agent_reference"}}
            while True:
                prompt = input("You (exit to stop): ").strip()
                if prompt.lower() in {"exit", "quit"}:
                    break
                if prompt:
                    print(finish_turn(client, conversation.id, reference, prompt))
        finally:
            try:
                if conversation is not None:
                    client.conversations.delete(conversation_id=conversation.id)
            finally:
                project.agents.delete_version(agent_name=agent.name,
                                              agent_version=agent.version)


if __name__ == "__main__":
    main()
```

The `version` in `agent_reference` keeps this session on the version just created. Use the same `conversation` for the initial response and every function result. The current exercise specifically calls out that mixing conversation state with a separate `previous_response_id` chain can leave a later turn without the expected function output.

## 4. Run and verify

```bash
python agent.py
```

Try these prompts in the same session:

- `Find the next event in South America and estimate five hours on a premium telescope with normal priority.`
- `What would that cost with high priority?`
- `Generate an observation report for that event, in South America, with five hours on a premium telescope, high priority, observer Alex.`

The pinned data use a premium rate of **$300/hour** and a normal multiplier of **1.25**, so the first estimate is **$1,875**. High priority is **1.75**, giving **$2,625**. The event answer varies with the current date; an empty result late in the year is valid. These expected values come from the [sample rate files](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/02-agent-custom-tools/Python/data).

Check that a follow-up can call a tool again, that the report exists in `reports`, and that an invalid tier or nonpositive hour count produces a validation error. A tool schema improves model output shape; it does not replace application validation or authorization.

## Cleanup and attribution

`exit` deletes the conversation and the agent version this script created. It does not delete other versions, your project, or model deployments. Remove generated reports when finished. Delete the dedicated lab project/resource group in Azure if you no longer need it and it contains only your lab resources.

Adapted from Microsoft Learn instructions and MicrosoftLearning sample code under the **MIT License**; see [Third-party notices](../../../THIRD_PARTY_NOTICES.md). The code was syntax-checked and its local functions exercised; a complete Azure run requires your subscription, deployed model, permissions, and credentials.
