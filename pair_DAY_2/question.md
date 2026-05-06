# Day 2 Question — Agent and Tool-Use Internals

## Final Question

In `orchestration/handoff.py`, `route_inbound_message()` receives a prospect's reply in `message.body` and routes it by checking membership in six hardcoded token lists: `_CURIOUS_TOKENS`, `_SOFT_DEFER_TOKENS`, `_PRICING_TOKENS`, `_OFFSHORE_CONCERN_TOKENS`, `_SCHEDULING_TOKENS`, and a three-token opt-out check for `"stop"`, `"unsubscribe"`, and `"unsub"` (lines 440–509). If none of those tokens match, the function falls through to the default scheduling branch and books a discovery call regardless of what the prospect actually said.

This means a reply of "Not interested, please remove me from your list" — which contains none of the six opt-out tokens — bypasses the opt-out branch entirely, qualifies the prospect, and returns `next_action="send_email"` with a booking confirmation. The prospect's intent is never read by the model. The routing decision is made entirely by Python string matching before any model call occurs.

**Specifically: what would a `classify_reply_intent` function-calling tool look like that reads `message.body` before routing runs — including the tool schema, the model call, and how the returned intent label gates the pipeline — and what is the model actually doing at the token level when it selects an intent label from a prospect's free-text reply?**

## Artifact Connection

This gap sits directly in `orchestration/handoff.py` at lines 427–581, where the entire reply routing logic lives. The function currently makes zero model calls — routing is deterministic string matching. Adding `classify_reply_intent` as a function-calling tool before line 440 would gate all six downstream branches on model-classified intent rather than token presence. To write that correctly in `method.md` and defend it in a code review, I need to understand what the model is mechanically doing when it maps "Not interested, please remove me" to an intent label — not just that it "understands" the reply, but what happens at the token level that produces a structured classification output.