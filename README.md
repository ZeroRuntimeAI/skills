# Zero Runtime Skills

Developer skills for building, customizing, running, debugging, and deploying Zero Runtime voice agents.

## Skills

- `zeroruntime-setup` - Choose an example, customize its pipeline and behavior, run it locally, smoke-test real voice behavior, and troubleshoot startup, audio, session, and telephony failures.
- `zeroruntime-deploy` - Authenticate, initialize deployment metadata, deploy, invoke, and verify an agent.

## Design

GitHub examples provide reusable product behavior. Pipelines default to the Zero Runtime Inference Gateway (`zeroruntime.inference`), which authenticates every stage with a single `ZERORUNTIME_AUTH_TOKEN`. When a developer pins a provider or model the gateway does not serve, only that stage falls back to its direct `zeroruntime.plugins` equivalent with its own API key; the rest stay on the gateway. The skills treat provider and model choices as overrides and use the `zeroruntime` CLI for execution and deployment. Both skills make MCP use mandatory: every provider, model, class name, environment variable, and CLI flag is verified against the MCP server before code is written or a command is run, because the tables in the skills are dated snapshots.

Gateway provider list: https://docs.zeroruntime.ai/plugins/inference/zero-runtime.md

Documentation index: https://docs.zeroruntime.ai/llms.txt

MCP server: https://mcp.zeroruntime.ai

Examples: https://github.com/ZeroRuntimeAI/zeroruntime-python-examples/

The setup skill uses the examples repository as a catalog. It maps developer intent to the smallest relevant example, then applies documented provider and feature overrides. It does not assume that the example's default pipeline is fixed.
