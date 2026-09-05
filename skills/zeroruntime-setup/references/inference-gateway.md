# Inference Gateway (Default Pipeline Source)

Source page: `https://docs.zeroruntime.ai/plugins/inference/zero-runtime.md`

The Inference Gateway routes STT, LLM, TTS, turn detection, and denoise through Zero Runtime
with a single auth token, so no per-provider API keys are needed. Gateway classes are drop-in
replacements for their `zeroruntime.plugins` equivalents.

Build every pipeline from the gateway by default. Fall back to a direct plugin only for the
specific stage the developer pinned to a provider or model the gateway does not serve.

## Gateway providers

| Stage | Gateway providers |
| --- | --- |
| Speech-to-text | Deepgram, Google, Sarvam AI, AssemblyAI, Cartesia |
| LLM | Google, Sarvam AI |
| Text-to-speech | Cartesia, Google, Deepgram, Sarvam AI |
| Turn detection | Echo (`echo-small`, `echo-large`) |
| Denoise | ai-coustics, Sanas |

VAD is not a gateway stage; `SileroVAD` stays a local plugin.

This table is a snapshot and changes as providers are added. Re-checking it against the MCP server
at `https://mcp.zeroruntime.ai` (`search_zero_runtime` or the documentation filesystem tool) is
mandatory before declaring a provider available or unavailable on the gateway.

## Imports

| Stage | Import |
| --- | --- |
| Speech-to-text | `from zeroruntime.inference import DeepgramSTT, GoogleSTT, SarvamAISTT, AssemblyAISTT, CartesiaSTT` |
| LLM | `from zeroruntime.inference import GoogleLLM, SarvamAILLM` |
| Text-to-speech | `from zeroruntime.inference import CartesiaTTS, GoogleTTS, DeepgramTTS, SarvamAITTS` |
| Turn detection | `from zeroruntime.plugins import TurnDetector` (Echo models route through the gateway) |
| Denoise | `from zeroruntime.inference import Denoise, AICousticsDenoise, SanasDenoise` |

`Denoise.sanas()` and `Denoise.aicoustics()` are factory equivalents of the named aliases.

## Default gateway pipeline

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

## Mixed pipeline (pinned stage falls back to a direct plugin)

Gateway and direct-provider components mix freely in one pipeline. Change only the pinned
stage and keep the rest on the gateway:

```python
from zeroruntime.inference import CartesiaTTS, SarvamAISTT   # gateway
from zeroruntime.plugins import OpenAILLM                    # direct: model not on the gateway

pipeline = Pipeline(
    stt=SarvamAISTT(),
    llm=OpenAILLM(model="<verified-model-id>"),  # needs its own API key in .env
    tts=CartesiaTTS(),
)
```

Verify the plugin's class name, constructor arguments, and required environment variables
against MCP before writing the direct-plugin code. This check is mandatory; never invent them
and never rely on the imports table above alone.

## Environment

- Gateway stages: `ZERORUNTIME_AUTH_TOKEN` only.
- Each direct-plugin stage: add that provider's documented key to `.env` and `.env.example`.
