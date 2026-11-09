# 04 How to integrate custom tools

[How to integrate custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/4-how-use-custom-tools?pivots=text)

The model chooses which function to request. Your application chooses which functions are allowed to run, executes them, and sends their outputs back. The next model response can request another function or answer the user. Hosted Azure Function and OpenAPI tools delegate execution to Azure instead, as shown in [unit 03](03-custom-tool-options.md#other-execution-options).

## Lab: astronomy agent

This completes the [Microsoft Learn custom-function exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html) using [`functions.py` from unit 02](02-why-use-custom-tools.md) and [`tools.py` from unit 03](03-custom-tool-options.md#describe-the-three-functions-to-the-model). The three lessons form one application: data and actions → tool schemas → execution loop.

Create or select a Foundry project, deploy the exercise's `gpt-5` model, and copy the project endpoint ending in `/api/projects/<project-name>`. Use your actual deployment name if it differs. From the `Labfiles/02-agent-custom-tools/Python` directory used in unit 02:

```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install "azure-ai-projects==2.3.0" "openai<3" azure-identity python-dotenv
az login
```

Create `.env` locally, using an identity with access to the project:

```dotenv
PROJECT_ENDPOINT=https://{resource-name}.services.ai.azure.com/api/projects/<project-name>
MODEL_DEPLOYMENT_NAME=gpt-5
```

### Match each call to its result

Start `agent.py` with this block. `FUNCTIONS` is the allowlist of executable functions. `finish_turn` keeps all requests and results on one conversation and checks whether the response is final before enforcing its eight-round limit.

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


from tools import TOOLS


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


```

The key is `call_id`: it associates each `function_call_output` with the exact call the model requested. Return only results from the latest response; resending old call IDs can corrupt the conversation. Expected JSON or argument errors become tool results, while a failed response or exhausted loop raises a visible application error. This avoids printing an empty answer when the model still needs a tool result.

### Connect the functions to the agent

Append this block to `agent.py`:

```python
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

`create_version` uses `agent_name=` in this SDK. `tools=TOOLS` publishes the contracts; it does not run the functions. Including the newly created `version` in `agent_reference` ensures this chat uses those exact contracts. The context managers close clients, and `finally` deletes this application's conversation and agent version even when a turn fails.

### Follow the result through the conversation

```bash
python agent.py
```

| Ask in the same session | What the code should do |
| --- | --- |
| Find the next event in South America and estimate five hours on a premium telescope with normal priority. | Read the event file and calculate **$1,875** from the sample rates |
| What would that cost with high priority? | Reuse the conversation context and calculate **$2,625** |
| Generate a report for that event, South America, five hours premium, high priority, observer Alex. | Create a file inside `reports/` and return its path |

Read the generated file and compare the numbers with the tool output. An event lookup can return no remaining sample event; the model should report that rather than inventing one. The program's cleanup removes its conversation and created version. The Foundry project and model deployment remain available for further lessons.
