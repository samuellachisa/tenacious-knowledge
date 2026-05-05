# Does Your Agent Recompute the Same Tokens 60 Times a Week?

*Written for Bethel's question: does prefix caching fire on my agent's API calls, what do I need to add, what silently breaks it, and what is the real dollar difference at 60 calls per week?*

---

You wrote a cost-per-qualified-lead number in your memo. That number is only defensible if you know whether your system prompt is being reprocessed from scratch on every API call or served from cache. Right now in `agent/openrouter_client.py`, no caching is configured. This explainer closes that gap.

## The Load-Bearing Mechanism: KV Cache and Prefix Caching

Every time you call an LLM, the model encodes your entire input through the attention mechanism. For every token in the input, it computes Keys and Values the data structures the model uses to figure out which tokens should attend to which. This is the **prefill phase**, and you pay for it on every call.

The **KV cache** stores those computed Keys and Values in memory so they do not have to be recomputed when generating each output token. Without it, generating a 200-token response would require recomputing the full input representation 200 times.

**Prefix caching** extends this idea across API calls. Instead of throwing away the KV cache at the end of each request, the provider stores it and reuses it on the next request that starts with the same prefix. If your system prompt is identical across calls which yours is the model can skip recomputing it entirely and read the cached state instead.

The economics are significant:
- **Cache write:** slightly above base input price (paid once per TTL window)
- **Cache read (hit):** roughly 10% of base input price a 90% discount
- **Cache miss:** full input price, same as today
- **TTL:** typically 5 minutes, refreshed on every cache hit at no cost

## Your Specific Situation

Looking at your `agent/openrouter_client.py`, the API call passes the system prompt as a plain string with no caching configuration:

```python
payload = {
    "model": model,
    "temperature": temperature,
    "response_format": {"type": "json_object"},
    "messages": [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt},
    ],
}
```

The good news: your `agent/email_generator.py` already keeps the system prompt fully static. Your dynamic content `first_name`, `company_name`, `signal_facts` all goes into `user_prompt`, not `system_prompt`. This means your system prompt is already in the right shape for caching. You just have not enabled it yet.

## How to Enable It Depends on Your Model

Since you use OpenRouter, the answer depends on which model `OPENROUTER_MODEL` is set to in your `.env`. Different models handle prefix caching differently:

**If routing to a Claude model (Anthropic):**
Caching is opt-in. You must add `cache_control` explicitly to the system message:

```python
"messages": [
    {
        "role": "system",
        "content": [
            {
                "type": "text",
                "text": system_prompt,
                "cache_control": {"type": "ephemeral"}
            }
        ]
    },
    {"role": "user", "content": user_prompt},
],
```

Verify it fired by checking the response:
```python
usage = data.get("usage", {})
print(usage.get("cache_read_input_tokens", 0))
print(usage.get("cache_creation_input_tokens", 0))
```

**If routing to a GPT-4o model (OpenAI):**
Caching is automatic for prompts over 1,024 tokens. No code change needed but you also have no explicit control. Check `usage.prompt_tokens_details.cached_tokens` in the response to verify.

**If routing to Qwen3-Next (which is Bethel's model):**
Prefix caching is not available on this model via OpenRouter, and the reason is architectural. Qwen3-Next uses a hybrid architecture combining standard Transformer attention with linear attention layers called Gated Delta Net. These linear attention layers have recurrent state dependencies, meaning the current state depends on all historical inputs. This makes traditional KV cache prefix caching inapplicable at the token level. The vLLM project tracks this as a known limitation and the official Qwen3-Next Usage Guide explicitly states the model does not support automatic prefix caching.

Three paths forward exist:

**Path 1 - Application-level response caching.** Cache complete API responses by hashing the system prompt and user prompt together. This skips the LLM call entirely for identical inputs. Useful for repeated qualification checks on the same prospect but does not help when the user prompt changes per prospect:

```python
import hashlib, json

_response_cache = {}

def chat_json(*, system_prompt, user_prompt, temperature=0.2):
    cache_key = hashlib.sha256(
        json.dumps([system_prompt, user_prompt]).encode()
    ).hexdigest()
    if cache_key in _response_cache:
        return _response_cache[cache_key]
    result = _actual_api_call(system_prompt, user_prompt, temperature)
    _response_cache[cache_key] = result
    return result
```

**Path 2 - Self-host via vLLM with the FP8 variant.** The FP8 variant of Qwen3-Next does support prefix caching when self-hosted with `--enable-prefix-caching`. This requires GPU infrastructure and is outside the challenge week budget but is the right answer for production deployment.

**Path 3 - Switch models for production.** Changing `OPENROUTER_MODEL` in `.env` to a Claude or GPT-4o model immediately unlocks native prefix caching with the exact code change shown above. This is the lowest-friction path and the most practical recommendation for a Tenacious production deployment.

**The first thing to do:** check what `OPENROUTER_MODEL` is set to before adding any code. The fix is different depending on the answer.

## What Silently Invalidates the Cache

Your codebase is already safe from the most common trap because dynamic content goes into `user_prompt`. But these patterns would silently break caching if introduced later:

```python
# BREAKS caching date changes every call
system_prompt = f"Today is {datetime.now().strftime('%B %d, %Y')}. You generate..."

# BREAKS caching prospect name in system prompt
system_prompt = f"You are writing to {prospect_name} at {company}..."

# SAFE your current pattern
system_prompt = "You generate concise B2B outbound sales emails..."
# all dynamic content stays in user_prompt ✓
```

Other silent invalidators: changing tool definitions between calls, adding or removing images anywhere in the prompt, and on Claude specifically changing `cache_control` placement between calls.

One more: there is a minimum token threshold before caching fires typically 1,024 tokens across providers that support it. Your current system prompt is approximately 60 tokens far below the minimum on any model. To benefit from caching you would need to include your static seed assets style guide, case study notes, email sequences in the system prompt rather than the user message. That pushes you well over the threshold and makes caching worthwhile regardless of which model you are on.

## The Real Dollar Difference at 60 Calls Per Week

Assume you extend the system prompt to include your style guide and case study notes, reaching approximately 1,200 tokens. Using a mid-tier model at $3.00 per million input tokens as a reference:

**Without caching today:**
```
60 calls × 1,200 tokens × $3.00/1M = $0.22/week
```

**With caching after the fix:**
```
Call 1 (cache write): 1,200 × $3.75/1M  = $0.0045
Calls 2–60 (cache reads): 59 × 1,200 × $0.30/1M = $0.0212
Total: ~$0.026/week
```

That is an **88% reduction** on system prompt tokens from $0.22 to $0.026 per week. At scale across hundreds of prospects this flows directly into your cost-per-qualified-lead number in the memo.

**TTL consideration:** At 60 calls across a 5-day week, roughly 12 calls per day. If calls run in morning batches the 5-minute TTL keeps the cache warm. If spread across the day with 40+ minute gaps between calls the cache expires and you pay full price each time.

## Adjacent Concepts Worth Knowing

**Check your model first.** The right implementation depends entirely on which model is behind `OPENROUTER_MODEL`. One environment variable lookup before touching any code saves you from adding `cache_control` to a model that ignores it or handles it automatically.

**Structured outputs compose cleanly.** Your `response_format: {"type": "json_object"}` setup is fully compatible with caching on models that support it.

**No manual eviction.** There is no API call to clear the cache. When you update your system prompt the old version expires naturally after the TTL.

**Cache hit rate is now measurable.** Once enabled, tracking `cache_read_input_tokens` across your eval runs gives you a real cache efficiency number something you have never measured before and which belongs in the cost section of your memo.

## Sources

See `sources.md`.