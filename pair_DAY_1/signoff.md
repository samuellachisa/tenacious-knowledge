# signoff.md

**Gap closure judgment: Fully closed**

---

Before this explainer, I understood that prefix caching existed as a concept but I had no idea whether it was actually firing on my agent's API calls. I was writing a cost-per-qualified-lead number in my memo without being able to defend the inference cost underneath it — I did not know whether my system prompt was being reprocessed from scratch on every call or served from cache, and I did not know what the difference would look like in dollars.

The explainer closed four things I genuinely did not know:

**First**, caching is not automatic or universal — it is model-dependent, and the right implementation is different for Claude (opt-in via `cache_control`), GPT-4o (automatic above 1,024 tokens), and Qwen3-Next (not supported at the token level due to its hybrid linear attention architecture). I would have added `cache_control` to my code and assumed it was working regardless of which model was behind `OPENROUTER_MODEL`. That would have been a silent failure.

**Second**, my system prompt at ~60 tokens is below the minimum threshold for caching to fire on any provider. I had no idea this threshold existed. This means even if I had configured caching correctly, I would have seen zero cache hits and not known why.

**Third**, there is a concrete dollar number I can now defend: extending the system prompt to ~1,200 tokens (by moving static assets like the style guide and case study notes into it) and enabling caching drops weekly system prompt cost from $0.22 to $0.026 — an 88% reduction. That number belongs in the memo and I can now derive it from first principles rather than guessing.

**Fourth**, I now know exactly what silently breaks the cache: injecting dynamic content like dates or prospect names into the system prompt rather than the user prompt. My current code is already safe from this, but I would not have known to protect it going forward.

The explainer is technically correct, grounded in my actual codebase, and gave me four concrete things to act on rather than a generic explanation of how caching works.