# AgentTest GitHub Action

AgentTest is a CI/CD cost guardrail for AI agents.

This public repository is intentionally a thin GitHub Action client. It runs inside your workflow, executes the configured command, collects cost-only usage metadata through the local `@agenttest/sdk` collector, signs it with a repository service key, and sends it to the private AgentTest API. It is installed directly from its repository; AgentTest is not pursuing a Marketplace listing in this phase.

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
      - uses: actions/checkout@<reviewed-immutable-sha>
      - uses: <GITHUB_ORG>/agenttest-action@v1
        with:
          service-key: ${{ secrets.AGENTTEST_SERVICE_KEY }}
          config-path: "./agenttest.config.yml"
          trigger-mode: "ready"
```

Store `AGENTTEST_SERVICE_KEY` only as a GitHub Actions secret.

## Configuration

Create `agenttest.config.yml` in your repository root:

```yaml
agent:
  # Safe metadata label. The command and prompt remain in this runner.
  name: sales-agent
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

Your command reads its test input from `AGENTTEST_PROMPT` and reports provider usage with
`@agenttest/sdk`. The SDK accepts only model/token/tool-count metadata and communicates only with
the short-lived collector on `127.0.0.1`.

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `service-key` | Yes | Repository-scoped service key, passed from `secrets.AGENTTEST_SERVICE_KEY`. |
| `api-key` | No | Deprecated compatibility alias for `service-key`. |
| `config-path` | No | Path to `agenttest.config.yml`. Defaults to `./agenttest.config.yml`. |
| `trigger-mode` | No | `ready`, `label`, or `always`. Use `ready` for first rollout. |

## Outputs

| Output | Description |
| --- | --- |
| `status` | `pass`, `fail`, `calibration`, or `skipped`. |
| `median-cost` | Median server-calculated cost returned by AgentTest. |
| `z-score` | Cost anomaly Z-score returned by AgentTest. |
