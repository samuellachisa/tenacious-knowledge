# Morning Call Summary — Day 4

The original question used technical language — "paired bootstrap", "variance reduction", "task-level difficulty variance" — before establishing why any of those terms mattered. The core confusion was identified as: Martha had implemented paired bootstrap correctly in `run_ablations.py` but could not explain why pairing helped, or whether it was strictly necessary for her 3pp Delta A target.

The question was sharpened by leading with the practical consequence first — a 3pp target that is statistically undetectable without pairing — and then naming the mechanism question: what is the mathematical property of pairing that makes the CI narrow enough to detect it. The artifact connection was tightened to point specifically to `run_ablations.py` line 77 (the `paired_bootstrap` function) and `methodology_rationale.md` (where the choice needs to be defended).

Both partners agreed the question is resolvable in one explainer: the answer requires understanding the covariance cancellation formula, showing the CI width difference numerically, and explaining why task-level difficulty variance in the benchmark is what makes the covariance large enough to matter.