# 02 Why use custom tools

[Why use custom tools](https://learn.microsoft.com/en-us/training/modules/build-agent-with-custom-tools/2-why-use-custom-tools?pivots=text)

- A custom tool is your function, your Azure Function, or your HTTP API. The agent decides when to call it from the user message.
- The exam reasons: automate a repeated step, return the same answer every time, and reach a system the model cannot see (CRM, inventory, scheduling, tickets, an LMS).
- Flow on the diagram: the user asks, the agent picks the tool, the tool returns data, the agent answers. You do not write the `if` that calls the tool.

The exercise function the agent is allowed to call:

```python
def next_visible_event(location: str) -> str:
    """Returns the next visible astronomical event for a location."""
    today = int(datetime.now().strftime("%m%d"))
    loc = location.lower().replace(" ", "_")
    for name, event_type, date, date_str, locs in EVENTS:
        if loc in locs and date >= today:
            return json.dumps({
                "event": name,
                "type": event_type,
                "date": date_str,
                "visible_from": sorted(locs),
            })
    return json.dumps({"message": f"No upcoming events found for {location}."})
```
