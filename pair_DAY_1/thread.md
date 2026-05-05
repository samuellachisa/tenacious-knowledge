# X Thread - Day 1

---

**Tweet 1**
Your outbound agent sends the same system prompt on every API call.

Are those tokens recomputed each time or cached?

If you have never checked cache_read_input_tokens, the answer is: recomputed. Every single time.

Here is what that costs 🧵

---

**Tweet 2**
Every LLM call runs two phases:

Prefill: reads your input, builds a KV cache. Full price.

Decode: generates output one token at a time from that cache.

Prefix caching saves your system prompt KV state across calls so the next request skips recomputing. 90% cheaper.

---

**Tweet 3**
Not all models support it:

Claude: add cache_control explicitly
GPT-4o: automatic above 1,024 tokens
Qwen3-Next: not supported

Why Qwen3-Next fails: hybrid linear attention (Gated Delta Net) has recurrent state dependencies. KV prefix caching is architecturally impossible.

---

**Tweet 4**
On Claude, the fix is switching your system message from a plain string to a content block with cache_control.

Then verify with cache_read_input_tokens in the response.

One catch: needs 1,024+ tokens. Prepend your static seed assets to hit the threshold.

---

**Tweet 5**
Real numbers. 60 calls/week, 1,200-token prompt:

Without caching: $0.22/week
With caching: $0.026/week

88% less. Flows into your cost per qualified lead.

Why: output tokens cost 4x more than input tokens. Small response length cuts save more than large prompt reductions.

---

**Tweet 6**
On Qwen3-Next? Three options:

1. Hash prompt pairs, cache full responses in memory
2. Self-host FP8 variant via vLLM with prefix caching enabled
3. Switch to Claude or GPT-4o, one env var change

Option 3 is the fastest path.

Full explainer: https://x.com/SamuelLachisa/status/2051722840643125303?s=20