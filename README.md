# Deploy Claude Agent

A GitHub Action to deploy
[Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview)
from version control.

## Why?

Managed agents are HTTP services. Like any other service, their configuration
should live in version control. The fact that instructions are written in prose
instead of code doesn't change this. Agents as code.

## Example

Deploys your agent whenever its config file changes on `main`.

### .github/workflows/deploy_agent.yml

```yaml
name: Deploy Claude Agent

on:
  push:
    branches: [main]
    paths: [agents/**]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy agent
        id: deploy
        uses: LukeLalor/claude-deploy@v1
        with:
          agent_id: ${{ vars.AGENT_ID }}
          config_file: agents/my-agent.yml
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

      - run: echo "Deployed version ${{ steps.deploy.outputs.version }}"
```

### agents/my-agent.yml

```yaml
name: My Agent
description: Does something useful
model:
  id: claude-opus-4-6
  speed: standard
system: You are a helpful assistant.
```

To bootstrap your config file from an existing agent:

```bash
curl -s https://api.anthropic.com/v1/agents/$AGENT_ID -H 'anthropic-version: 2023-06-01' -H 'anthropic-beta: managed-agents-2026-04-01' -H "X-Api-Key: $ANTHROPIC_API_KEY" | yq -p json -o yaml 'del(.version)' > agents/my-agent.yml
```

## Inputs

| Input               | Required | Description                                                      |
| ------------------- | -------- | ---------------------------------------------------------------- |
| `agent_id`          | Yes      | The ID of the Claude agent to update                             |
| `config_file`       | Yes      | Path to the agent YAML config file (relative to repository root) |
| `anthropic_api_key` | Yes      | Anthropic API key with permission to manage agents               |

## Outputs

| Output    | Description                        |
| --------- | ---------------------------------- |
| `version` | The agent version after the update |

## Pinning a version

By default the action fetches the current version from the API automatically and
passes it through with the update. To control versioning yourself, include
`version` explicitly in your config file. Note that update operations will fail
if the version is out of date, so opting in to this means you are responsible
for keeping it in sync.

```yaml
name: My Agent
model:
  id: claude-opus-4-6
  speed: standard
system: You are a helpful assistant.
version: 4
```
