# Sources Day 1

## Canonical Sources

**Source 1**
Anthropic. *Prompt Caching Claude API Documentation.*
https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching

Load-bearing for: the cache_control parameter syntax, the pricing structure (1.25x write, 0.10x read), the 5-minute TTL, the minimum token threshold of 1,024 tokens for Sonnet models, and the complete list of cache invalidation conditions.

**Source 2**
Pope et al. (2023). *Efficiently Scaling Transformer Inference.* Google Research.
https://arxiv.org/abs/2211.05102

Load-bearing for: the prefill and decode phase distinction, why the KV cache exists, and why decode is memory-bandwidth bound and sequential while prefill is compute-bound and parallel.

## Engineering Sources

**Source 3**
vLLM Project. *Feature Request: Enable Automatic Prefix Caching for Qwen3-Next (hybrid attention).* GitHub Issue #25874.
https://github.com/vllm-project/vllm/issues/25874

Load-bearing for: the confirmed statement that Qwen3-Next does not support automatic prefix caching, tracked as an open feature request.

**Source 4**
xllm Project. *Support Prefix Cache for Qwen-Next Model (Mamba Cache Mode).* GitHub Issue #1016.
https://github.com/jd-opensource/xllm/issues/1016

Load-bearing for: the architectural explanation of why prefix caching is inapplicable to Qwen3-Next. The linear attention (Gated Delta Net) layers have recurrent state dependencies that make traditional KV cache prefix caching impossible at the token boundary level.

**Source 5**
vLLM Recipes. *Qwen3-Next Usage Guide.*
https://docs.vllm.ai/projects/recipes/en/latest/Qwen/Qwen3-Next.html

Load-bearing for: the confirmation that the FP8 variant of Qwen3-Next supports prefix caching when self-hosted via vLLM with --enable-prefix-caching, as a workaround for the OpenRouter limitation.

## Tool Used

**Anthropic Tokenizer**
https://platform.openai.com/tokenizer

Used to verify that a representative system prompt expanded with style guide and case study assets exceeds the 1,024-token minimum threshold. The dollar calculation in the explainer was computed against verified token counts.

## Follow-on Reading

Kwon et al. (2023). *Efficient Memory Management for Large Language Model Serving with PagedAttention.* SOSP 2023. https://arxiv.org/abs/2309.06180 explains KV cache memory layout and why prefix reuse is architecturally possible in standard transformer models but requires special handling in hybrid recurrent architectures.