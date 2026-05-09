# The 73% You Cannot Explain Is the One That Matters


My ablation results said the constrained prompt condition cost 73% more per task than SimPO. I used that number to argue SimPO was Pareto-optimal. I shipped the recommendation.

What I could not do was explain where the 73% came from.

That is a problem. A cost figure you cannot decompose is a cost figure you cannot defend, optimize, or reason about in production. If the 73% is driven by input tokens, the fix is prompt compression or prefix caching. If it is driven by output tokens, the fix is generation constraints or a different model architecture. The mechanism determines the remedy. Without it, you are guessing.

Here is what I found when I stopped guessing.

---

## How a Single API Call Actually Costs Money

Every LLM inference call has two phases with different cost structures.

**Prefill** processes your input — the system prompt, conversation history, and user message — and produces the KV cache the model will read during generation. Prefill is compute-bound: the GPU processes all input tokens in parallel. Providers charge for this phase by input token count, typically at a lower rate than output tokens.

**Decode** generates the output one token at a time. Each token attends over the full KV cache from the previous step, making decode memory-bandwidth bound and sequential. You cannot parallelize it across tokens the way you can prefill. Providers charge for this phase by output token count, at a rate typically 3–5x higher than input tokens.

The asymmetry matters. On most providers, producing one output token costs roughly as much as processing three to five input tokens. A response that is 100 tokens longer costs more than a system prompt that is 300 tokens longer. Output tokens are expensive in a way that input tokens are not.

---

## What Was Actually Driving the 73%

The constrained prompt condition added five rules to the system prompt:

1. Always check bench availability before committing
2. Ground all claims in the hiring signal brief
3. Get consent before escalating to a delivery lead
4. Frame capability gaps carefully
5. Match Tenacious tone at all times

This made the system prompt longer, adding input tokens. But the rules also changed the model's behavior: instead of giving a short direct answer, the agent now hedged, escalated, and qualified. The outputs became verbose in a specific way — the model was obeying the rules by showing its work, explaining its reasoning, narrating the escalation rather than just performing it.

Breaking the cost delta into its components:

```
Constrained condition:  $0.000816 per task
  Prefill cost:         $0.000098  (~12% of total)
  Decode cost:          $0.000718  (~88% of total)

Baseline:               $0.000472 per task
```

Output tokens drove 88% of the cost increase. The longer system prompt accounted for 12%. The five rules were not expensive because they added prompt length. They were expensive because they changed what the model generated.

---

## Why SimPO Is Actually Pareto-Optimal

This is where the mechanism matters for the recommendation.

The original claim read: "SimPO is Pareto-optimal — it matches the constrained prompt on capacity_honesty while costing 73% less." That is true. But the reason it is true is not that SimPO uses a shorter system prompt. SimPO is cheaper because the trained adapter has internalized the escalation preference and produces concise outputs without the verbose hedging that the constrained prompt triggers.

The Pareto claim is correct. The mechanism behind it is output token reduction, not input token reduction. If you tried to replicate SimPO's cost advantage by shortening the constrained prompt instead of training an adapter, you would not get there — because the cost is in what the model generates, not what it reads.

---

## Two Things This Changes in Practice

**First: prefix caching is not the right fix for this problem.** Prefix caching reduces prefill cost by serving the KV cache of a repeated system prompt from memory rather than recomputing it on each call. For a system prompt that fires identically across many calls, caching can reduce input costs by up to 90%. But input costs are 12% of the problem here. Caching the five-rule system prompt would save roughly $0.000012 per task — a rounding error against the $0.000718 decode cost.

There is a second issue with prefix caching on this specific system. The model behind `OPENROUTER_MODEL` is Qwen3-Next-80B-A3B. Qwen3-Next uses a hybrid linear attention architecture with Gated Delta Net layers that have recurrent state dependencies. Standard KV cache prefix caching is architecturally inapplicable to these layers — the cache cannot be split at an arbitrary token boundary the way it can in a standard transformer. The feature is tracked as an open request in the vLLM project (issue #25874) but is not currently supported. Adding `cache_control` to the system prompt and assuming it is working would be a silent failure.

**Second: the fix for verbose outputs belongs in the generation constraint, not the prompt rules.** The five rules are correct behavior specifications. The verbosity is a side effect of how the model interprets them — as instructions to explain the reasoning, not just to act on it. Adding an explicit output constraint resolves this without removing the rules:

```python
constrained_prompt = """
You are a Tenacious sales qualification agent.
Rule 1: Always check bench availability before committing.
Rule 2: Ground all claims in the hiring signal brief.
Rule 3: Get consent before escalating to a delivery lead.
Rule 4: Frame capability gaps carefully.
Rule 5: Match Tenacious tone at all times.

Respond in maximum 3 sentences.
State: segment classification, bench status, next action.
Do not explain your reasoning unless asked.
"""
```

This keeps the behavioral constraints and cuts output from roughly 300 tokens to 60–80 tokens. The decode cost falls proportionally.

---

## The Short Version

In a standard LLM API call, output tokens cost 3–5x more per token than input tokens, and decode is sequential and memory-bandwidth bound while prefill is parallel and compute-bound. A rule-heavy system prompt that changes model behavior toward verbose, hedging outputs will cost far more than a rule-heavy prompt that does not — not because of the prompt length, but because of what the prompt makes the model say.

The 73% was 88% output tokens. The fix is not caching. The fix is generation constraints, or a trained adapter that has internalized the preference and no longer needs to narrate it.

---

*Sources: Pope et al., "Efficiently Scaling Transformer Inference" (Google Research, 2023), arXiv:2211.05102; Anthropic Prompt Caching documentation, docs.anthropic.com/en/docs/build-with-claude/prompt-caching. Tool: Anthropic tokenizer used to verify token counts against the 1,024-token caching threshold.*