# Example Routing

Use the official examples repository as a living catalog:

`https://github.com/ZeroRuntimeAI/zeroruntime-python-examples/`

Pipelines default to the Inference Gateway (`zeroruntime.inference`); see `inference-gateway.md`. The routing below selects product behavior, not the provider source.

| Developer intent | Start with | Examples to inspect |
| --- | --- | --- |
| First voice agent | `getting_started` | `cascade_basic.py`, `realtime_basic.py`, `cascade_advanced.py` |
| Choose or understand pipeline slots | `pipelines` | `composable/agent_multimodal.py`, `hybrid/*`, `fallback_recovery.py` |
| Default gateway pipeline, one Zero Runtime credential | `inference` | `zeroruntime_cascade.py`, `zeroruntime_realtime.py` |
| Add actions or external services | `tools` | `cascade_tool_chaining.py`, `mcp_example.py`, `human_in_the_loop/*` |
| Memory, language, or handoffs | `context` | `agent_memory.py`, `handoffs/*`, `translator_agent.py` |
| Interruptions or speech behavior | `speech` | `reply_interrupt_agent.py`, `utterance_handle_agent.py`, `cached_tts.py` |
| Phone call behavior | `telephony` | `call_transfer.py`, `dtmf_voicemail.py`, `agent_hangup.py` |
| Logs, traces, or recording | `observability` | `pipeline_events.py`, `*pipeline_hooks.py`, `observability_hooks.py` |
| Camera or screen input | `vision` | `vision_cascade.py`, `vision_realtime.py` |
| Talking avatar | `avatars` | `avatar_simli_cascade.py`, `avatar_anam_cascade.py` |

When an example builds its pipeline from direct `zeroruntime.plugins` providers, move the unpinned stages to their `zeroruntime.inference` equivalents and keep only pinned, non-gateway models on direct plugins.

This routing table is a snapshot of the examples repository. Confirm the file still exists and still
demonstrates what you expect by reading it, and query MCP (`https://mcp.zeroruntime.ai`) for the
current API surface before copying code out of it.

Each file is standalone and begins with a comment explaining its point. Read that comment and any folder README before changing code. Preserve the selected example's behavior and customize only the requested slots.

For a customer-support request, the correct starting file depends on the requested behavior: use a tools or human-in-the-loop example when support actions are needed, a context handoff example when escalation is the main concern, and a getting-started example only for a simple conversational assistant.