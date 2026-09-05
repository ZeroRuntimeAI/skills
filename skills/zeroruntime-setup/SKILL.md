---
name: zeroruntime-setup
description: >
  Build, customize, run, and troubleshoot Zero Runtime voice agents. Use when a developer asks to
  create a voice agent, select an example, change STT, LLM, TTS, VAD, turn detection, denoise, agent
  instructions, tools, or greetings; to install the SDK, configure authentication, run an agent
  locally, or smoke-test audio; or when an agent does not start, hear, respond, register, or receive
  calls. Pipelines default to the Zero Runtime Inference Gateway; pinned providers or models outside
  the gateway use direct plugins.
license: MIT
metadata:
  author: Zero Runtime
  version: "0.2.0"
---

# Set Up a Zero Runtime Agent

Use this skill as the full developer loop: choose an example, customize it, run it locally, verify
real voice behavior, and debug what fails. Deployment is a separate skill (`zeroruntime-deploy`) and
runs only after this loop passes.

The developer's requested use case selects the example; the developer's requested pipeline overrides
the example defaults.

## Source of truth

- Examples: `https://github.com/ZeroRuntimeAI/zeroruntime-python-examples/`
- Documentation index: `https://docs.zeroruntime.ai/llms.txt`
- MCP server: `https://mcp.zeroruntime.ai`
- Inference Gateway providers and imports: `https://docs.zeroruntime.ai/plugins/inference/zero-runtime.md`, summarized in `references/inference-gateway.md`
- Example routing table: `references/example-routing.md`
- Failure triage: `references/troubleshooting.md`

## MCP is mandatory

Using the Zero Runtime MCP server at `https://mcp.zeroruntime.ai` is **required**, not optional. Query
it for current context *before* writing or changing any Zero Runtime code, and again whenever a fact
you are about to rely on could have changed.

Your training data is stale. Plugin class names, gateway provider coverage, model IDs, constructor
arguments, environment-variable names, and CLI flags change between releases. Never answer from
memory and never invent an API surface.

Query MCP before you:

- name or import any `zeroruntime.inference` or `zeroruntime.plugins` class,
- claim a provider or model is or is not served by the Inference Gateway,
- write constructor arguments, model IDs, or pipeline combinations,
- state a required environment variable or authentication method,
- run a `zeroruntime` CLI command whose flags you have not just verified,
- assert that something is unsupported.

How to use it:

- `search_zero_runtime` for discovery, then the documentation filesystem tool for exact page content.
- Read the relevant MCP resources when applicable: `agent-creation`, `pipeline-configuration`, and `tools-and-agent-extensions`.
- The MCP server is read-only. Use the local workspace and the `zeroruntime` CLI for edits and execution.
- The tables in this skill and in `references/` are **snapshots that go out of date**. Treat MCP as the
  source of truth and the tables as a starting hypothesis to confirm.

If the MCP server is unreachable, say so explicitly, fall back to the documentation index at
`https://docs.zeroruntime.ai/llms.txt`, and tell the developer which facts you could not verify.
Do not silently proceed on remembered API details.

## Default pipeline source

Build pipelines from the Zero Runtime Inference Gateway (`zeroruntime.inference`) by default. Gateway
components are drop-in replacements for their `zeroruntime.plugins` equivalents and authenticate with
`ZERORUNTIME_AUTH_TOKEN` alone, so no per-provider API keys are needed.

Gateway coverage (re-verify against the source page or MCP before calling anything unavailable):

| Stage | Gateway providers |
| --- | --- |
| Speech-to-text | Deepgram, Google, Sarvam AI, AssemblyAI, Cartesia |
| LLM | Google, Sarvam AI |
| Text-to-speech | Cartesia, Google, Deepgram, Sarvam AI |
| Turn detection | Echo (`echo-small`, `echo-large`) |
| Denoise | ai-coustics, Sanas |

Use a direct plugin only when the developer pins a specific provider or model that the gateway does
not serve for that stage. Then swap that stage alone, keep every other stage on the gateway, and add
that provider's documented key to `.env` and `.env.example`. Say explicitly which stages run through
the gateway, which run direct, and which extra credential the direct stage requires.

Do not silently move a stage off the gateway, and do not move a stage onto the gateway when the
developer pinned a provider or model the gateway does not serve.

## Required workflow

1. Clarify the use case, integration mode, pipeline mode, and required capabilities. If the developer has not explicitly specified the integration mode, ask:

  ```text
  Where do you want to integrate this agent?

  1. Playground
  2. Console
  3. Telephony
  ```

  If the developer chooses `Telephony`, ask a follow-up:

  ```text
  Which telephony direction do you need?

  1. Inbound: callers dial your number and reach the agent.
  2. Outbound: the agent places calls to a target number.
  ```

  Do not ask these questions when the developer already specified a mode, such as "build this for console" or "create an inbound telephony agent". Carry the selected mode and, when applicable, direction through example selection, implementation, local testing, and verification.
2. Route the request to the closest example category before choosing a file: `getting_started` for the first complete agent, `pipelines` for a pipeline shape, `inference` for gateway credentials, `tools` for actions or external services, `context` for memory/handoffs/language, `speech` for speaking and interruption control, `telephony` for phone behavior, `observability` for tracing/recording, `vision` for camera input, and `avatars` for a visual face. Inspect the available GitHub examples and choose the closest file. Preserve reusable business behavior such as tools, escalation, and conversation flow.
3. Inspect the selected example before editing. Identify its entry point, pipeline construction, `Agent` subclass, environment template, dependencies, and local run command.
4. Treat provider and model choices as configuration overrides on a gateway-default pipeline. Keep example defaults for components the developer did not request to change, but move unpinned stages to their `zeroruntime.inference` equivalents when the example uses direct plugins with no requested reason.
5. Resolve each requested provider or model against the gateway first. If the gateway serves it, import that stage from `zeroruntime.inference`. If it does not, use the documented direct plugin for that stage only. Before writing provider-specific code, you must query MCP for current plugin names, models, constructor arguments, supported pipeline combinations, streaming support, and required environment variables. Never invent these facts and never rely on the snapshot tables alone. For a request such as "pure Sarvam", verify Sarvam independently for STT, LLM, and TTS; only call it pure when all requested stages are supported and compatible.
6. Explain the proposed result and any compatibility limitation. If a full requested pipeline is unsupported, offer a supported mixed pipeline or stop before editing; do not silently substitute providers.
7. Apply the smallest changes needed to the example. Keep credentials in `.env` or the runtime secret store, and update `.env.example` without real values.
8. Set up the environment and run the agent locally (see **Run locally**), then complete the smoke test.
9. Summarize changed behavior, required environment variables, verification results, and the exact next command.

## Example selection

Use the smallest example that demonstrates the requested behavior. Do not use `getting_started/cascade_basic.py` as a generic replacement for a tools, telephony, or observability request when a focused example exists. Read the first comment in the selected file and any folder README before editing; each example is intentionally standalone.

When the developer requests a provider change, preserve the example's behavior and replace only the requested pipeline slot. For example, a customer-support workflow can come from `tools/human_in_the_loop/customer_agent.py` while STT, LLM, and TTS are replaced with documented Sarvam plugins.

## MCP resolution protocol

Record the result of provider resolution before editing:

```yaml
requested_pipeline: cascade
stt: {status: supported | unsupported | unknown, source: gateway | direct, reason: default | pinned_model}
llm: {status: supported | unsupported | unknown, source: gateway | direct, reason: default | pinned_model}
tts: {status: supported | unsupported | unknown, source: gateway | direct, reason: default | pinned_model}
compatibility: supported | partial | unknown
required_environment: []   # ZERORUNTIME_AUTH_TOKEN, plus one key per direct stage
```

Set `source: gateway` unless the developer pinned a provider or model that the gateway does not serve for that stage; then set `source: direct` with `reason: pinned_model`.

If any stage is `unknown`, retrieve the relevant documentation page before proceeding. If the result is `partial`, present the mixed pipeline explicitly and ask for confirmation. If it is `unsupported` on both the gateway and the direct plugins, stop before editing or offer a documented custom integration path.

## Pipeline decision

For cascade, preserve the example's audio controls unless requested and replace only the requested stages. Default every stage to the gateway; VAD stays a local plugin:

```python
from zeroruntime import Pipeline
from zeroruntime.inference import AICousticsDenoise, CartesiaTTS, GoogleLLM, SarvamAISTT
from zeroruntime.plugins import SileroVAD, TurnDetector

pipeline = Pipeline(
    stt=SarvamAISTT(),        # via gateway
    llm=GoogleLLM(),          # via gateway
    tts=CartesiaTTS(),        # via gateway
    vad=SileroVAD(),          # local plugin
    turn_detector=TurnDetector(model="echo-large"),  # Echo, via gateway
    denoise=AICousticsDenoise(model_id="quail-vf-2.2-l-16khz"),
)
```

When a pinned model is not on the gateway, swap that one stage to its direct plugin and leave the rest on the gateway. See `references/inference-gateway.md` for imports and the mixed-pipeline shape.

For realtime, use the documented realtime model as the pipeline's LLM/model component and do not combine it with cascade STT/TTS unless the documentation explicitly supports that combination.

## Agent behavior

Customize `agent_id`, instructions, tools, `on_enter`, and `on_exit` only after understanding the use case. Verify that the agent is served with the class, not a shared instance, when the SDK requires per-session state:

```python
zeroruntime.serve(MyVoiceAgent, on_ready=invoke_agent)
```

## Run locally

1. Verify Python 3.11+ is available. Prefer a project virtual environment.
2. For the official examples repository, prefer `uv sync`; otherwise install the SDK with the current documented command (`uv pip install zeroruntime` or `pip install zeroruntime`).
3. Read the complete current example's dependency file and its folder README, then install any additional dependencies or required services.
4. Create `.env` from `.env.example`. Never print or commit secret values.
5. Verify the current authentication variables and runtime target against Zero Runtime documentation or MCP. At minimum, the project needs a runtime target and auth token, or the documented API-key/secret alternative.
6. Because pipelines default to the Inference Gateway, `ZERORUNTIME_AUTH_TOKEN` covers every gateway stage. Require a provider API key only for a stage that was deliberately pinned to a direct plugin; a missing provider key for a gateway stage means the code, not the environment, is wrong.

Use the example's documented command. Common paths are:

```bash
uv run getting_started/cascade_basic.py
uv run getting_started/realtime_basic.py
```

Use `python getting_started/cascade_basic.py` equivalents inside an activated pip virtual environment. The examples normally print a playground URL; use the documented console mode only when that example supports it. Do not assume the entry-point filename or mode flags if the example says otherwise.

## Smoke test

Test real behavior in this order:

1. Confirm the process starts without import or authentication errors.
2. Confirm the agent registers or joins the expected room.
3. Confirm the greeting is audible.
4. Ask a normal request and confirm speech is transcribed and answered.
5. Interrupt a response and confirm the documented interruption behavior.
6. Exercise one requested tool or business workflow.
7. Exercise one unsupported or invalid request.
8. Stop the session cleanly and inspect errors.

For cascade, verify STT -> LLM -> TTS audio flow. For realtime, verify the speech-to-speech model path instead of expecting separate STT/TTS logs. In a mixed pipeline, confirm both the gateway stages and each direct-plugin stage in the logs.

## Troubleshoot

Start with evidence from the failing command and logs. Do not change providers or pipeline components before locating the failure stage. Follow the triage order and symptom table in `references/troubleshooting.md`.

On failure, capture the command, concise error, and relevant logs. Check environment activation, dependency versions, microphone permissions, credentials, provider configuration, and room/client matching before changing application code.

## Completion criteria

Do not call the task complete until the selected example starts locally, the requested conversation path is exercised, and any changed provider path is confirmed by logs or a real session. Confirm that gateway stages authenticate with `ZERORUNTIME_AUTH_TOKEN` and that each direct-plugin stage has its own key present. Only then offer the `zeroruntime-deploy` skill.
