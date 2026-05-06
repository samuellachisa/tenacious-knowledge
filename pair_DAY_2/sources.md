# Sources

## Canonical Sources

**1. Anthropic Tool Use Documentation**
https://docs.anthropic.com/en/docs/tool-use

The authoritative primary source for what Pattern 1 (model-driven tool use) looks like at the API level. Documents the `tool_use` and `tool_result` content block types, the `tools` parameter in the messages API, and the `tool_choice` field. This is the spec the explainer's Pattern 1 code example is drawn from. Used to establish the precise technical meaning of "model using tools" — a meaning the main pipeline does not satisfy.

---

**2. Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022)**
https://arxiv.org/abs/2210.03629

The paper that named and formalized the plan-act-observe loop that Pattern 1 implements. ReAct interleaves reasoning traces with action calls, where the model's reasoning output determines which tool is called next and what arguments it receives. Reading it makes concrete how far a fixed-sequence orchestrator is from what the literature means by an agent using tools. The gap between ReAct and TheConversionEngine is not a matter of degree — the orchestrator does not implement the loop at all in the main pipeline. Cited in the explainer to give the question its historical lineage and to show the concept has a precise research definition, not just an API definition.

---

## Tool or Pattern Used

**Direct source reading of TheConversionEngine**
`git clone https://github.com/NuryeNigusMekonen/TheConversionEngine.git`

Files read and traced:
- `agent/orchestration/service.py` — `run_toolchain` method, full tool execution sequence, `_with_retries` wrapper, `_handle_tool_result` failure logging
- `agent/generation/service.py` — `_chat_completion` payload (no `tools` key), `_messages_for_scenario` system prompt ("Rewrite the provided safe scaffold"), `draft_email_from_scaffold` try/except with deterministic fallback
- `agent/policies/service.py` — `draft_initial_decision`, risk flag assembly, `needs_human` derivation
- `agent/schemas/conversation.py` — `ConversationDecision` schema, `needs_human: bool` field
- `agent/orchestration/handoff.py` — token-matching logic (`_SMS_OPT_IN_TOKENS`, `_PRICING_TOKENS`), state machine, hand-authored reply templates

Every technical claim in the explainer is traceable to a specific function or line in one of these files. This is not inference about what the system probably does — it is direct reading of what the system does.

---

## Follow-On Reading

- **Anthropic MCP specification** — https://modelcontextprotocol.io/docs — for understanding how tool descriptions are structured so that a model uses them well (the adjacent question the explainer flags at the end)
- **Guidance library (Microsoft)** — https://github.com/guidance-ai/guidance — a concrete implementation of constrained decoding, relevant to the follow-on direction about how structured JSON output is enforced at the token level and what that means for the generation service's fallback behavior