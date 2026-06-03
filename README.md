# AgentTest GitHub Action

AgentTest is a CI/CD cost guardrail for AI agents.

This public repository is intentionally a thin GitHub Action client. It runs inside your GitHub Actions workflow, executes the agent command you configure, collects usage metadata, signs that metadata with your AgentTest repository API key, and sends the signed metadata to the private AgentTest backend API.

AgentTest's product source code, private API service, web app, cloud deployment code, tests, internal documentation, pricing logic, authorization, billing, data storage, and operational controls are not part of this public Action repository.

## Data Boundary

The Action is designed around this boundary:

- Customer prompts stay in customer CI.
- Customer completions stay in customer CI.
- Customer source code stays in customer CI.
- Customer LLM provider keys stay in customer CI.
- AgentTest receives signed telemetry metadata.

## Usage

```yaml
name: AgentTest

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, labeled]

jobs:
  cost-guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: <GITHUB_ORG>/agenttest-action@v1
        with:
          api-key: ${{ secrets.AGENTTEST_API_KEY }}
          config-path: "./agenttest.config.yml"
          trigger-mode: "ready"
```

Store `AGENTTEST_API_KEY` only as a GitHub Actions secret.

## Configuration

Create `agenttest.config.yml` in your repository root:

```yaml
agent:
  command: "npm run agent"
  prompt: "Summarize the latest sales report"
  framework: langchain
  primary_model: gpt-4o

limits:
  max_tokens_per_run: 500000
  max_duration_sec: 300
  num_runs: 5

server:
  endpoint: https://api.agenttest.dev
```

Your agent command must emit JSON usage metadata to stdout. AgentTest passes `--prompt <prompt> --agenttest-output json` to the command.

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `api-key` | Yes | AgentTest repository API key, passed from `secrets.AGENTTEST_API_KEY`. |
| `config-path` | No | Path to `agenttest.config.yml`. Defaults to `./agenttest.config.yml`. |
| `trigger-mode` | No | `ready`, `label`, or `always`. Use `ready` for first rollout. |
| `slack-webhook` | No | Optional Slack webhook for client-side notifications. |
| `jira-api-token` | No | Optional Jira token for client-side ticket creation. |
| `jira-email` | No | Optional Jira account email. |
| `jira-domain` | No | Optional Jira domain. |
| `jira-project-key` | No | Optional Jira project key. |

## Outputs

| Output | Description |
| --- | --- |
| `status` | `pass`, `fail`, `calibration`, or `skipped`. |
| `median-cost` | Median server-calculated cost returned by AgentTest. |
| `z-score` | Cost anomaly Z-score returned by AgentTest. |
