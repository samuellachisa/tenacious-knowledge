# Day 1 Question — Inference-time Mechanics

## Final Question

In `ablations/ablation_results.json`, the cost-quality analysis shows the constrained prompt condition costs **$0.000816 per task** versus the baseline condition at **$0.000472 per task** — a 73% increase. The repo uses this difference to argue that SimPO is Pareto-optimal, but I cannot explain where the 73% cost increase actually comes from.

The constrained prompt adds a 5-rule system prompt to the baseline call covering bench check, signal grounding, consent, gap framing, and tone. This means the constrained condition has more input tokens than baseline. But the constrained outputs are also more structured and verbose — the agent now hedges, escalates, and qualifies rather than giving a short direct answer — which means more output tokens too.

**Specifically: in a single LLM API call, how do input tokens (prefill) and output tokens (decode) each contribute to the total cost, and given that the constrained prompt adds both a longer system prompt and produces longer outputs, which of the two — the extra input tokens or the extra output tokens — is the dominant driver of the 73% cost increase between the baseline and constrained conditions in my evaluation?**

## Artifact Connection

This gap sits directly in `ablations/ablation_results.json` under `cost_quality_analysis`, where the Pareto-optimality claim for SimPO rests on the cost delta between conditions. The claim reads: *"Constrained prompt wins on raw overall pass@1 but costs 73% more per task than SimPO."* I use this number to recommend SimPO for deployment, but I cannot decompose it into its input and output token components — meaning the recommendation rests on a cost figure I cannot defend mechanically.

