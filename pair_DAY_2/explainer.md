# Who Is Actually Using the Tools? Orchestrator-Driven Pipelines vs. Model-Driven Tool Use

*When is it technically correct to say the model is using tools, and when is the orchestrator using tools while the model only rewrites or reviews content?*

---

## The Short Answer

In TheConversionEngine, the orchestrator is using the tools. The model is a rewriter and a binary gate. Those are meaningfully different things, and conflating them produces documentation that overclaims, failure analysis that looks in the wrong place, and evaluation that measures the wrong thing.

---

## Where Planning Actually Lives

**Model-driven tool use** has a precise meaning in the APIs. The model receives tool schemas — function names, parameter types, descriptions — as part of its context. When it decides a tool is needed, it emits a `tool_use` content block naming the function and its arguments. The host executes the function and feeds results back. The model's output determines what runs next. This is what Anthropic's tool_use API and the ReAct paper implement. The model is the planner.

TheConversionEngine does none of this. Reading `orchestration/service.py` directly: `run_toolchain` calls enrichment connectors, then `email_channel.send`, then `handoff_manager.prepare_warm_sms_handoff`, then `calcom_client.book_preview`, then `hubspot_client.record_conversation_event` — in that order, unconditionally, in Python. The OpenRouter call in `generation/service.py` sends only `model`, `messages`, and `temperature`. No `tools` parameter. The model has never been told functions exist.

**The planning lives entirely in Python.** The orchestrator is the planner. The model is a transformation step inside a slot the orchestrator already defined.

---

## Three Patterns, Often Conflated

**Pattern 1 — Model-as-planner.** Model receives tool schemas, emits tool call syntax, determines what executes next. This is what "agent using tools" technically means.

**Pattern 2 — Orchestrator-as-planner, model-as-transform.** Python sequences the calls. The model rewrites content inside a slot. It cannot change what runs next. `draft_email_from_scaffold` is this pattern: the model returns better JSON than the fallback scaffold, nothing more.

**Pattern 3 — Model-as-reviewer, gating one branch.** The model returns a typed verdict. The orchestrator acts on one field. `ConversationDecision.needs_human` is this pattern: the model sets a boolean, the orchestrator routes accordingly.

TheConversionEngine uses Pattern 2 in the generation layer and Pattern 3 in the policy layer. Pattern 1 does not appear anywhere in the main pipeline.

Here is what that difference looks like in code:

```python
# TheConversionEngine — Pattern 2
# No tools key. Model is a text transformer.
payload = {
    "model": settings.openrouter_model,
    "messages": messages,
    "temperature": 0.2,
}

# Pattern 1 — model is the planner
response = anthropic.messages.create(
    model="claude-sonnet-4-6",
    tools=[{"name": "send_email", "input_schema": {...}}],
    messages=[{"role": "user", "content": prompt}]
)
# Model emits tool_use block → host executes it
```

In Pattern 1, `send_email` runs because the model asked for it. In TheConversionEngine, `email_channel.send` runs because a line in `orchestration/service.py` says so.

---

## Failures Split Across This Boundary

Because planning lives in the orchestrator and content lives in the model, failures fall into three non-overlapping surfaces:

**Orchestrator failures** — wrong sequence, bad state transition, integration error. If HubSpot sync fires before email send, that is an orchestrator bug. Reproducible without any model involvement. Debuggable with unit tests.

**Model failures** — bad content quality within a slot. Draft too aggressive, subject line too long, hallucinated detail. These require eval against a golden set or a judge model, not a stack trace. The `validate_email_style` check guards the most common cases with automatic fallback.

**Governance failures** — the policy layer passed or blocked the wrong draft. `draft_initial_decision` sets `needs_human` from risk flags. If that verdict is wrong, it requires its own eval, separate from generation quality.

These three surfaces need different tests, different evals, and different Langfuse queries. Langfuse captures all three, but you have to know which surface you are querying.

---

## Why the Proactive Tool-Use Instruction Mattered for tau2-bench but Not Here

In tau2-bench, the model is given real tool schemas and scored on whether it emits write-action tool calls to complete retail tasks. A model that describes what it would do without emitting tool call syntax scores zero. The proactive instruction fixed that — it told the model: you have tools, use them now.

In the sales pipeline, the instruction is irrelevant. The model is never in a planning role. No tool schemas exist to invoke. No amount of instruction changes what the orchestrator calls next. The same model plays different roles in different contexts. The confusion arises when documentation collapses all three patterns into the word "agent."

---

## What to Fix in `method.md` and `README.md`

The accurate one-sentence description: *all tool execution is sequenced by the orchestrator; the model rewrites content in a fixed slot and sets a governance flag.*

The one place you can honestly say "model-influenced control flow" is `needs_human` in the reply path. That is not planning. It is a model setting one boolean inside a typed schema. Call it a governance gate, not tool selection.

---

## Pointers

- Anthropic tool use documentation: [docs.anthropic.com/en/docs/tool-use](https://docs.anthropic.com/en/docs/tool-use) — the authoritative spec for Pattern 1, including `tool_use` and `tool_result` content block types
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), arXiv:2210.03629 — the paper that formalized the plan-act-observe loop; reading it makes clear how far a fixed-sequence orchestrator is from what the literature means by an agent
- Follow-on: **constrained decoding** — how the JSON the generation service parses is enforced at the token level, and what that means for fallback behavior when the model drifts