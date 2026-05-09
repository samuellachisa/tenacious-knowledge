# Grounding Commit — Day 2

## What Changed

**Files edited:** `orchestration/handoff.py` and `method.md` in `TheConversionEngine`

**Specific edit:** The architecture description in `method.md` was corrected to use the three-pattern framework accurately, and a `classify_reply_intent` design spec was added documenting the function-calling tool that would fix the intent classification gap in `route_inbound_message()`.

## Before (`method.md` architecture section)

```
The Tenacious Conversion Engine is an AI agent that uses tools to qualify
prospects, send personalized outreach, and manage the full sales pipeline.
The agent reasons over prospect signals and selects the appropriate tool
for each stage of the engagement.
```

## After (`method.md` architecture section)

```
The Tenacious Conversion Engine is an orchestrator-controlled pipeline.
All tool execution is sequenced by Python. The model plays two roles:
a content rewriter (Pattern 2) that generates personalized email drafts
inside a fixed slot, and a governance gate (Pattern 3) that returns a
typed `needs_human: bool` verdict gating the human-escalation branch.
The model does not receive tool schemas, does not emit tool call syntax,
and does not determine execution order. Planning lives in the orchestrator.

The one place model output influences control flow is the `needs_human`
flag in `policies/service.py` — this is Pattern 3 (model-as-reviewer),
not Pattern 1 (model-as-planner). The distinction matters for debugging:
orchestrator failures are Python sequencing bugs; model failures are
content quality failures; governance failures are verdict-layer failures.
Each requires different instrumentation to surface.
```

## Design spec added to `method.md` (`classify_reply_intent`)

```python
# Proposed addition to orchestration/handoff.py
# Insert before line 440 (token-matching logic)

async def classify_reply_intent(message_body: str) -> str:
    """
    Classify a prospect reply into one of six intent labels before
    token-matching routing runs. Uses function-calling so the model
    emits a structured label rather than free text.
    """
    response = anthropic.messages.create(
        model="claude-haiku-4-5",
        max_tokens=64,
        tools=[{
            "name": "set_intent",
            "description": "Set the classified intent of a prospect reply.",
            "input_schema": {
                "type": "object",
                "properties": {
                    "intent": {
                        "type": "string",
                        "enum": [
                            "opt_out", "curious", "soft_defer",
                            "pricing", "offshore_concern", "scheduling"
                        ]
                    }
                },
                "required": ["intent"]
            }
        }],
        tool_choice={"type": "tool", "name": "set_intent"},
        messages=[{"role": "user", "content": message_body}]
    )
    tool_block = next(b for b in response.content if b.type == "tool_use")
    return tool_block.input["intent"]
```

The model selects a label by computing log-probabilities over the constrained
output space defined by the `enum`. Constrained decoding at the token level
means the model cannot output a label outside the six defined values.
The returned label then gates all six downstream branches instead of the
current token-list matching.

## Why This Edit

`route_inbound_message()` makes zero model calls. A prospect replying
"Not interested, please remove me from your list" routes to the scheduling
branch and receives a booking confirmation because none of the six opt-out
token lists match. The design spec documents the fix that would close this
gap in v2: model-classified intent before routing runs. The `method.md`
correction ensures the architecture is described accurately, which matters
for any code review or FDE handoff of this system.