# Day 3 Question — Training and Post-Training Mechanics

## Final Question

In `training_data/pairs.jsonl`, all 200 SimPO training pairs were built by `programmatic_template_v1` — slot-filled templates with fixed chosen and rejected response structures. Reading the data directly: 200 pairs share only 82 unique input contexts, meaning 118 pairs reuse the same input with different template slots filled. The chosen responses across all 200 pairs follow the same escalation-language pattern. The rejected responses follow the same hard-commitment pattern. Both come from the same template pool in `training_data/generate_pairs.py`.

The SimPO loss operates on token log-probabilities. If chosen and rejected responses share large token subsequences — which template pairs nearly always do — the gradient signal concentrates on the few differing tokens rather than on the underlying behavioral preference. This means the adapter may have learned to prefer specific template vocabulary rather than a generalizable preference for escalation over commitment.

**Specifically: when a SimPO adapter is trained on preference pairs built from templates with limited input diversity — 82 unique contexts across 200 pairs — what is the difference between an adapter that has genuinely learned the behavioral preference and one that has memorized surface-form patterns, and what would you look for in the training process and eval results to tell them apart?**

## Artifact Connection

This gap sits directly in `training_data/pairs.jsonl` and affects the Pareto-optimality claim in `ablations/ablation_results.json`, where the SimPO adapter achieves 82% capacity_honesty. If those gains come from template pattern matching rather than genuine preference learning, they will not hold on out-of-distribution inputs where prospects use phrasing the templates never covered. Closing this gap would let me add a concrete data quality section to `methodology.md` explaining what the 82 unique input contexts actually test and what they leave uncovered, and design a paraphrase robustness check for the v2 training data.