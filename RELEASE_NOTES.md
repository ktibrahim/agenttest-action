# AgentTest Action Release Notes

## v1

- Public thin-client GitHub Action bundle for AgentTest.
- Runs the configured agent command in customer CI.
- Sends only signed telemetry metadata to the private AgentTest backend API.
- Keeps prompts, completions, source code, and provider API keys in customer CI.
- Shares its execution engine with `@agenttest/cli` and receives usage through `@agenttest/sdk`.
- Removes client-side notification credentials; destinations are managed server-side.
- Distributed directly without a GitHub App or Marketplace listing.
