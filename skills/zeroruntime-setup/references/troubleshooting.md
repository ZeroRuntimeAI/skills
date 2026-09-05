# Troubleshooting

Start with evidence from the failing command and logs. Do not change providers or pipeline
components before locating the failure stage.

Querying the MCP server at `https://mcp.zeroruntime.ai` is mandatory before you attribute a failure to
a provider, class name, environment variable, or CLI flag. The symptom list below is a snapshot;
confirm the current behavior against MCP rather than from memory.

## Triage order

1. Reproduce the failure with the smallest command.
2. Check Python/SDK/CLI versions and the active environment.
3. Check `.env` variable names without exposing values.
4. Read local output or `zeroruntime logs`.
5. Identify the stage: startup, authentication, registration, STT, turn detection, LLM, TTS, room/session, deployment, or SIP routing.
6. Make one focused change and rerun the same check.

## Common symptoms

- Import errors: verify the active virtual environment and current SDK installation.
- Authentication errors: verify runtime target, token or API-key/secret method, and provider keys. Gateway stages (`zeroruntime.inference`) need only `ZERORUNTIME_AUTH_TOKEN`; a per-provider key error from a gateway stage usually means an invalid or expired auth token, and a direct-plugin stage that errors is missing its own key.
- Provider not served by the gateway: confirm the stage against the gateway provider table in `inference-gateway.md` before assuming a credential problem; an unsupported provider belongs on a direct plugin, not the gateway.
- No greeting: verify `serve()` wiring, agent registration, and session entry behavior.
- No transcription: verify microphone permissions, STT configuration, and audio input.
- No response: verify turn detection, LLM configuration, and logs.
- No output audio: verify TTS configuration, output permissions, and audio format support.
- Deployment inactive: run `zeroruntime version status` and inspect `zeroruntime logs`.
- Stuck session: run `zeroruntime session list`, then stop only the intended room/session.
- Telephony failure: verify the running agent's exact `agent_id`, SIP gateway addresses, provider configuration, and routing rule.

## Escalation

When the documented integration or CLI behavior is unclear, query the MCP server at
`https://mcp.zeroruntime.ai` before guessing. If the issue is a product defect,
preserve the reproduction command, versions, redacted configuration shape, and relevant log excerpt.
