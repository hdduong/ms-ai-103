# 07 Maintain Workflows in Microsoft Foundry

[Maintain Workflows in Microsoft Foundry](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/7-maintain-workflows?pivots=text)

- The canvas and the YAML are the same workflow. Edit either one and the other updates.
- Every save creates a new immutable version. You can compare and roll back.
- Notes on a node are for the next person: why the branch exists, what the variable holds.
- Before you ship a change, drop unused nodes and check that structured agent output is still what the next node reads.

## List the saved versions

A workflow is an agent of kind workflow, so the agents API lists its versions. Each save adds one, and the name in code stays the same.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project_client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint="https://{resource-name}.services.ai.azure.com/api/projects/<project-name>",
)
for version in project_client.agents.list_versions(
    agent_name="ContosoPay-Customer-Support-Triage",
):
    print(version.version, version.created_at)
```

`get_version(agent_name, agent_version)` loads one version to compare before a rollback.
