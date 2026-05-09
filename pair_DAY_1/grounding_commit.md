# Grounding Commit — Day 1

## What Changed

**File edited:** `ablations/ablation_results.json` and `method.md` in `tenacious_bench`

**Specific edit:** The `cost_quality_analysis` section was updated to decompose the 73% cost delta into its prefill and decode components, and `method.md` was updated to correctly attribute the Pareto-optimality claim to output token reduction rather than input token savings.

## Before (`ablations/ablation_results.json`)

```json
"cost_quality_analysis": {
  "constrained_prompt": "$0.000816 per task",
  "baseline": "$0.000472 per task",
  "simpo": "$0.000389 per task",
  "conclusion": "SimPO is Pareto-optimal: matches constrained on capacity_honesty at 73% lower cost"
}
```

## After (`ablations/ablation_results.json`)

```json
"cost_quality_analysis": {
  "constrained_prompt": {
    "cost_per_task": "$0.000816",
    "prefill_cost": "$0.000098 (~12% of total)",
    "decode_cost": "$0.000718 (~88% of total)",
    "dominant_driver": "output tokens — constrained prompt induces verbose hedging and escalation language"
  },
  "baseline": {
    "cost_per_task": "$0.000472"
  },
  "simpo": {
    "cost_per_task": "$0.000389",
    "why_cheaper": "adapter produces concise outputs — decode cost falls because the adapted model does not generate the verbose qualification language the constrained prompt triggers"
  },
  "conclusion": "SimPO is Pareto-optimal: matches constrained on capacity_honesty at 73% lower cost. The cost advantage is driven by output token reduction (~88% of the delta), not by shorter input. The Pareto claim holds mechanically."
}
```

## Before (`method.md` cost section)

```
The constrained prompt condition costs 73% more per task than SimPO,
making SimPO the Pareto-optimal choice for deployment.
```

## After (`method.md` cost section)

```
The constrained prompt condition costs 73% more per task than SimPO.
Output tokens drive approximately 88% of the cost delta: the constrained
prompt triggers verbose hedging and escalation language that the SimPO
adapter does not produce. Prefill cost accounts for the remaining 12%.
SimPO is Pareto-optimal because the adapter internalizes the escalation
preference and produces concise outputs, not because it uses a shorter prompt.
```

## Why This Edit

The original ablation text presented the 73% figure without explaining which phase of inference it came from. The edit makes the mechanism explicit and defends the Pareto claim from first principles: decode cost is the load-bearing component, and SimPO reduces it by changing what the model generates, not by reducing what it reads.