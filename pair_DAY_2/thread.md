# Medium Post — Who Is Actually Using the Tools?

**Subtitle:** A precise answer to a question most AI engineers get wrong

**Tags:** AI Engineering, LLMs, Agents, Software Architecture, Machine Learning

---

Your sales agent has tools. It enriches leads, sends emails, syncs HubSpot, books meetings. You call it an agent.

But here is the question worth pausing on: when a tool runs, who actually decided to run it — the model, or your Python code?

This distinction is not semantic. It determines where planning lives, how failures get debugged, and whether the word "agent" in your README is accurate or an overclaim.

---

## The precise technical meaning of "model using tools"

Model-driven tool use has a specific meaning at the API level. The model receives tool schemas — function names, parameter types, descriptions — as part of its context. When it decides a tool is needed, it emits a `tool_use` content block naming the function and its arguments. The host reads that block, executes the function, and feeds the result back. The model's output determines what runs next.

This is what Anthropic's tool_use API and the ReAct paper (Yao et al., 2022) formalize. The model is the planner. Control flow is a function of model output.

Most production pipelines do not do this.

---

## What orchestrator-driven pipelines actually look like

In an orchestrator-driven pipeline, Python code determines the sequence. The model is invoked at a fixed point to rewrite content or return a verdict. It cannot change what runs next.

Here is what that looks like in code:

```python
# Orchestrator-driven — model is a text transformer
payload = {
    "model": settings.openrouter_model,
    "messages": messages,
    "temperature": 0.2,
    # No tools key. The model has never been told functions exist.
}

# vs. model-driven tool use
response = anthropic.messages.create(
    model="claude-sonnet-4-6",
    tools=[{"name": "send_email", "input_schema": {...}}],
    messages=[{"role": "user", "content": prompt}]
)
# Model emits tool_use block → host executes it
```

In the first case, `send_email` runs because a line in the orchestrator says so. In the second, it runs because the model asked for it. The difference is who holds the plan.

---

## Three patterns, one word that collapses them all

Systems that involve a model and external tools fall into three distinct patterns:

**Pattern 1 — Model-as-planner.** The model receives tool schemas, emits tool call syntax, and its output determines what executes next. This is what "agent using tools" technically means.

**Pattern 2 — Orchestrator-as-planner, model-as-transform.** Python sequences the calls. The model rewrites content inside a slot the orchestrator already defined. The model cannot change what runs next.

**Pattern 3 — Model-as-reviewer, gating one branch.** The model returns a typed verdict — a boolean, a label, a risk flag. The orchestrator acts on it. The model influences one downstream branch through a narrow interface, not open-ended planning.

The word "agent" gets applied to all three. That is where the overclaiming starts.

---

## How failures split across this boundary

Because planning lives in the orchestrator and content lives in the model, failures do not mix — they split cleanly into three non-overlapping surfaces.

**Orchestrator failures** are sequencing bugs, bad state transitions, or integration errors. If HubSpot syncs before email sends, that is a Python bug. Reproducible without any model involvement. Debuggable with unit tests.

**Model failures** are content quality failures within a slot. The draft is too aggressive. The subject line is too long. The rewrite hallucinated a detail. These require eval against a golden set or a judge model. A stack trace will not find them.

**Governance failures** are verdict-layer failures. The policy model passed a draft it should have flagged, or flagged a clean one. These need their own eval, separate from generation quality, because the judge is itself a model-influenced component with its own failure modes.

Each surface needs different instrumentation. Langfuse tracing captures all three, but you need different query patterns to surface each type.

---

## Why a proactive tool-use instruction can matter in one context and be irrelevant in another

This is the part that confuses people who build both benchmarks and production pipelines.

In a benchmark like tau2-bench, the model is given real tool schemas and scored on whether it emits write-action tool calls to complete tasks. A model that describes what it would do without emitting tool call syntax scores zero. A proactive instruction — "you have tools, use them, do not just describe" — fixes this by correcting the model's self-understanding of its role.

In an orchestrator-driven pipeline, that same instruction is irrelevant. The model is never in a planning role. No tool schemas exist to invoke. No amount of prompting changes what the orchestrator calls next.

The same model can play all three patterns in the same codebase, in different contexts. The confusion arises when documentation collapses all three into the word "agent."

---

## What to actually write in your README

The accurate description is:

*All tool execution is sequenced by the orchestrator. The model rewrites content in a fixed slot and sets a governance flag. It does not select tools, does not determine execution order, and does not produce tool call syntax.*

The one place you can honestly say "model-influenced control flow" is wherever a model output gates a downstream branch through a typed schema — a boolean, an enum, a risk flag list. That is Pattern 3. Name it accurately as a governance gate, not tool selection.

That framing is not weaker than calling the system an agent. It is more defensible, more precise, and more useful to anyone who has to maintain or evaluate the system after you.

---

## Pointers

- Anthropic tool use documentation: [docs.anthropic.com/en/docs/tool-use](https://docs.anthropic.com/en/docs/tool-use) — the authoritative spec for Pattern 1, including `tool_use` and `tool_result` content block types
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), arXiv:2210.03629 — the paper that formalized the plan-act-observe loop that Pattern 1 implements
- Follow-on worth reading: constrained decoding — how structured JSON output from a model call is enforced at the token level, which explains why Pattern 2's fallback behavior works the way it does