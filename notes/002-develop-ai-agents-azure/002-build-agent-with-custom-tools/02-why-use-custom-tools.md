# 02 Why use custom tools

[Why use custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/2-why-use-custom-tools?pivots=text)

A custom tool lets an agent retrieve data or perform an operation that its model cannot perform by generating text alone. The model chooses a tool and proposes arguments; your application runs a local function and returns its result. A deterministic calculation belongs in code, while the model handles the user's wording and explains the result.

The [Microsoft Learn astronomy exercise](https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/02-agent-custom-tools.html) illustrates three reasons to add tools:

| User request | Python function | Why a tool is useful |
| --- | --- | --- |
| Find an event visible from South America | `next_visible_event` | Reads the supplied event data |
| Estimate five hours of premium telescope time | `calculate_observation_cost` | Applies explicit rates and arithmetic |
| Write an observation report | `generate_observation_report` | Creates a local file |

The functions below complete the exercise's `functions.py`. [Unit 03](03-custom-tool-options.md#describe-the-three-functions-to-the-model) describes them to the model, and [unit 04](04-how-use-custom-tools.md#lab-astronomy-agent) connects them to an agent. Keep these files in the same working directory.

## Get the sample data

Use Python 3.13, the version specified by the exercise. Clone the source once; if you already have this checkout, open the indicated directory instead. The pinned [starter and data files](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/4d6a32629722927bd5f1034e0b5396e0da4e962c/Labfiles/02-agent-custom-tools/Python) match these examples.

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
cd Labfiles/02-agent-custom-tools/Python
```

## Read the data and find an event

Replace `functions.py` with the following block, then append the next two blocks to that same file. `BASE` makes data paths relative to the file, so the functions do not depend on the terminal's current directory.

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


```

The model supplies a continent such as `south_america`. Python normalizes that value, finds matching events on or after today's month and day, and returns the earliest match as JSON. These are illustrative events with no year, not a live astronomy feed; no match late in the year is a valid result.

## Calculate instead of guessing

Append this function to `functions.py`:

```python
def calculate_observation_cost(telescope_tier: str, hours: float,
                               priority: str) -> str:
    tier = telescope_tier.strip().lower()
    priority = priority.strip().lower()
    if tier not in TELESCOPE_RATES:
        return json.dumps({"error": "Unknown telescope tier."})
    if priority not in PRIORITY_MULTIPLIERS:
        return json.dumps({"error": "Unknown priority."})
    if (isinstance(hours, bool) or not isinstance(hours, (int, float))
            or not math.isfinite(hours) or hours <= 0):
        return json.dumps({"error": "Hours must be a positive finite number."})
    rate = TELESCOPE_RATES[tier]
    multiplier = PRIORITY_MULTIPLIERS[priority]
    base = rate * hours
    return json.dumps({
        "telescope_tier": tier, "hours": hours, "hourly_rate": rate,
        "priority": priority, "priority_multiplier": multiplier,
        "base_cost": base, "total_cost": base * multiplier,
    })


```

The calculation is `hourly rate × hours × priority multiplier`. For the sample premium rate of $300 and normal multiplier of 1.25, five hours costs **$1,875**. Changing only the priority to high (1.75) gives **$2,625**. Python checks the inputs even though the model will also receive a schema: the schema describes acceptable arguments, while the function protects its own calculation. Expected validation failures return an error in JSON so the agent can ask for a correction.

## Return an artifact from an action

Append the final function to `functions.py`:

```python
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

A report is an action with a visible result: a file path in the returned JSON. This adaptation keeps filenames inside `reports/`, sanitizes the model-supplied event name, and adds a unique suffix. It does not make a telescope reservation or a payment. The same pattern applies to a support ticket or database update: return the operation's actual result, rather than asking the model to claim that it succeeded.

You can check the calculation locally before connecting any model:

```python
from functions import calculate_observation_cost

print(calculate_observation_cost("premium", 5, "normal"))
```

The output includes `"total_cost": 1875.0`. Next, [describe these functions as tools](03-custom-tool-options.md#describe-the-three-functions-to-the-model).
