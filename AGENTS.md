# Zero Runtime Skills

These skills follow the Agent Skills format.

## Principles

- Use GitHub examples as starting templates, not immutable implementations.
- Keep durable workflow guidance in skills.
- Querying the MCP server (`https://mcp.zeroruntime.ai`) for current context is mandatory before writing code or running CLI commands. Verify SDK, provider, model, environment-variable, and CLI facts there; never answer from memory.
- Treat every table in a skill or reference as a dated snapshot, and say so in the text, so the model confirms it against MCP instead of trusting it.
- Default pipelines to the Inference Gateway (`zeroruntime.inference`); fall back to a direct plugin only for a stage pinned to a provider or model the gateway does not serve.
- Never place real credentials in source files.
- Test real agent behavior, not only process startup.
- Keep each `SKILL.md` concise and focused; push long tables and symptom lists into `references/`.

## Repository Structure

```text
zeroruntime-skills/
├── README.md
├── AGENTS.md
└── skills/
    ├── zeroruntime-setup/       # build, customize, run, smoke-test, troubleshoot
    │   └── references/
    │       ├── example-routing.md
    │       ├── inference-gateway.md
    │       └── troubleshooting.md
    └── zeroruntime-deploy/      # authenticate, deploy, invoke, verify
```
