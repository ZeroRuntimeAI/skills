---
name: zeroruntime-deploy
description: >
  Deploy and verify Zero Runtime agents with the zeroruntime CLI. Use when a developer asks to authenticate,
  initialize, deploy, invoke, inspect versions, upload secrets, or take an agent offline.
license: MIT
metadata:
  author: Zero Runtime
  version: "0.2.0"
---

# Deploy a Zero Runtime Agent

Deploy only after the `zeroruntime-setup` workflow passes: the agent runs locally and its real voice
behavior is verified.

## MCP is mandatory

Using the Zero Runtime MCP server at `https://mcp.zeroruntime.ai` is **required**, not optional. Query
it for current context before running any `zeroruntime` command whose behavior you have not just
verified.

CLI command names, flags, deployment metadata, and secret-handling behavior change between releases,
and a wrong flag here mutates a real deployment. Never answer from memory. The commands shown in this
skill are a **snapshot**: confirm them against MCP, or `zeroruntime <command> --help`, before running
them.

If the MCP server is unreachable, say so explicitly, fall back to `--help` and the documentation index
at `https://docs.zeroruntime.ai/llms.txt`, and tell the developer which facts you could not verify.

## Prerequisites

1. Run `zeroruntime --version` and verify the installed CLI.
2. Authenticate with `zeroruntime auth login` when needed; browser authentication requires the developer.
3. Confirm the project root, entry point, and `zeroruntime.yaml` state.
4. Confirm a Dockerfile/build configuration exists if the current deployment path requires it.
5. Confirm `.env` contains required provider keys, but do not print values.

Verify current flags and behavior with MCP or `zeroruntime <command> --help` before running commands that may have changed.

## Deployment flow

For a new agent:

```bash
zeroruntime init --name <agent-name>
zeroruntime up --env .env
```

If the project already has deployment metadata, inspect it and avoid reinitializing without a reason. Explain whether `zeroruntime up` will create a new immutable version.

## Verify deployment

Run the narrowest checks in sequence:

```bash
zeroruntime version status
zeroruntime version list
zeroruntime invoke
zeroruntime logs --limit 200 --sort 1
```

Use `zeroruntime invoke --console` when supported and appropriate. Verify the agent registers, becomes active, starts a session, speaks its greeting, answers one normal request, and logs no provider or authentication failure.

## Secrets

Use `zeroruntime up --env .env` or the documented secrets commands. Never paste API-key values into chat, source, YAML, or logs. Report only secret names or whether the upload succeeded.

## Completion report

Report the agent name, deployed version/status, verification session result, log result, and any remaining manual action. Do not claim success from a completed upload alone; the agent must be invoked and behavior-checked.

When a deployment fails to become active, register, or receive calls, use the triage order in the
`zeroruntime-setup` skill (`references/troubleshooting.md`) before changing the pipeline.
