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
# Get the lab files (skip if you already cloned them)
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents
# Pin the commit whose files match these examples
git checkout 4d6a32629722927bd5f1034e0b5396e0da4e962c
# Starter code and data files for this lab
cd Labfiles/02-agent-custom-tools/Python
```

## Read the data and find an event

Replace `functions.py` with the following block, then append the next two blocks to that same file.

```python
import json
import math
import re
from datetime import datetime
from pathlib import Path
from uuid import uuid4

# This file's folder, so data paths work from any directory
BASE = Path(__file__).resolve().parent


def load_rates(filename):
    # Each line is name|value, e.g. premium|300.00
    rates = {}
    for line in (BASE / "data" / filename).read_text().splitlines():
        key, value = line.split("|")
        rates[key] = float(value)
    return rates


# Read once at import: hourly rate per tier, multiplier per priority
TELESCOPE_RATES = load_rates("telescope_rates.txt")
PRIORITY_MULTIPLIERS = load_rates("priority_multipliers.txt")


def next_visible_event(location: str) -> str:
    # Today as MMDD (Nov 25 is 1125) so dates compare as numbers
    today = int(datetime.now().strftime("%m%d"))
    # "South America" becomes south_america, the form the data uses
    normalized = location.strip().lower().replace(" ", "_")
    candidates = []
    # Each line: name|type|MM-DD|continent;continent
    for line in (BASE / "data/events.txt").read_text().splitlines():
        name, event_type, date, locations = line.split("|")
        month, day = map(int, date.split("-"))
        visible_from = locations.split(";")
        # Keep events visible from here, dated today or later
        if normalized in visible_from and month * 100 + day >= today:
            candidates.append((month * 100 + day, {
                "event": name, "type": event_type, "date": date,
                "visible_from": sorted(visible_from),
            }))
    # Return the soonest match as JSON text for the model
    if candidates:
        return json.dumps(min(candidates, key=lambda item: item[0])[1])
    # No match is a normal answer, not an error
    return json.dumps({"message": f"No remaining sample event for {location}."})


```

The model supplies a continent such as `south_america`. These are illustrative events with no year, not a live astronomy feed; no match late in the year is a valid result.

## Calculate instead of guessing

Append this function to `functions.py`:

```python
def calculate_observation_cost(telescope_tier: str, hours: float,
                               priority: str) -> str:
    # Ignore case and stray spaces in what the model sends
    tier = telescope_tier.strip().lower()
    priority = priority.strip().lower()
    # Validate even with a schema; a JSON error lets the agent ask again
    if tier not in TELESCOPE_RATES:
        return json.dumps({"error": "Unknown telescope tier."})
    if priority not in PRIORITY_MULTIPLIERS:
        return json.dumps({"error": "Unknown priority."})
    # Positive finite number only; bool counts as int, so reject it first
    if (isinstance(hours, bool) or not isinstance(hours, (int, float))
            or not math.isfinite(hours) or hours <= 0):
        return json.dumps({"error": "Hours must be a positive finite number."})
    # Cost = hourly rate * hours * priority multiplier
    rate = TELESCOPE_RATES[tier]
    multiplier = PRIORITY_MULTIPLIERS[priority]
    base = rate * hours
    # Return each input and step so the model can explain the total
    return json.dumps({
        "telescope_tier": tier, "hours": hours, "hourly_rate": rate,
        "priority": priority, "priority_multiplier": multiplier,
        "base_cost": base, "total_cost": base * multiplier,
    })


```

For the sample premium rate of $300 and normal multiplier of 1.25, five hours costs **$1,875**. Changing only the priority to high (1.75) gives **$2,625**.

## Return an artifact from an action

Append the final function to `functions.py`:

```python
def generate_observation_report(event_name: str, location: str,
                                telescope_tier: str, hours: float,
                                priority: str, observer_name: str) -> str:
    # Reuse the cost function; pass its error back if inputs are bad
    cost = json.loads(calculate_observation_cost(telescope_tier, hours, priority))
    if "error" in cost:
        return json.dumps(cost)
    # Add the next sample event for this location to the report
    next_event = json.loads(next_visible_event(location))
    # Reports go in a reports/ folder beside this file
    reports = BASE / "reports"
    reports.mkdir(exist_ok=True)
    # event_name comes from the model; keep only safe filename characters
    safe_name = re.sub(r"[^a-z0-9_-]+", "_", event_name.lower()).strip("_")[:60]
    # Unique suffix so a new report doesn't overwrite an old one
    target = reports / f"report_{safe_name or 'observation'}_{uuid4().hex[:8]}.txt"
    # Plain-text report from the results; no booking or payment happens
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
    # Return what actually happened: the path of the file written
    return json.dumps({"status": "Report generated", "file": str(target)})
```

A report is an action with a visible result: a file path in the returned JSON. It does not make a telescope reservation or a payment. The same pattern applies to a support ticket or database update: return the operation's actual result, rather than asking the model to claim that it succeeded.

You can check the calculation locally before connecting any model:

```python
from functions import calculate_observation_cost

# Premium, 5 hours, normal priority: 300 * 5 * 1.25 = 1875
print(calculate_observation_cost("premium", 5, "normal"))
```

The output includes `"total_cost": 1875.0`. Next, [describe these functions as tools](03-custom-tool-options.md#describe-the-three-functions-to-the-model).
