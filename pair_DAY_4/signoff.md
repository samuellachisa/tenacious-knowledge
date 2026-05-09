# Sign-off — Day 4

**Gap closure status: Closed**

Before reading the explainer, I understood that the Automaton Auditor's scores varied across runs — I had observed a 2.30-point range across 15 runs on the same repo — but I was treating this as an implementation quirk rather than a statistical problem. I was reporting single scores in audit reports without any indication that those numbers were one sample from a distribution, not a stable measurement.

After reading Samuel's explainer, I now have two tools I did not have before and a concrete number I can act on immediately.

The ICC framing gave me the right conceptual anchor: the fraction of total variance that is between targets versus within targets. When MSW dominates — which a 2.30-point spread on a single target implies — ICC approaches zero, and the rank ordering the system produces is more noise than signal. I had not thought about the scoring system's variance in terms of what it does to rank ordering, only in terms of what it does to absolute score values. That reframing is more useful.

The SEM table made the problem concrete in score units. Under a generous ICC assumption of 0.50, a reported score of 3.00 truthfully covers a range of 2.57 to 3.43 — a range that straddles most pass/fail thresholds I would use in practice. That is not a rounding error; it is an interval wide enough to flip a hiring or code-quality decision. I was presenting numbers with that level of uncertainty as if they were measurements.

The minimum-runs calculation answered the question I had not known to ask. Nine runs is the threshold for a 95% CI within ±0.5 points, given the observed SD of 0.611. The 15 runs I already had are more than sufficient — the problem is they were never aggregated. The fix is not more data; it is reporting the mean and CI from the data I already have.

What I am changing in `audit/report_onself_generated/` and `README.md`: the audit output now reports the mean score and 95% CI across multiple runs rather than a single point estimate. The README documents that any single-run score should be interpreted with a ±0.5 uncertainty at 95% confidence, and explains that 9 runs is the minimum for a stable mean.