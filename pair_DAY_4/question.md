# Day 4 Question — Evaluation and Statistics

## Final Question

The Automaton Auditor runs three LLM judges — Prosecutor, Defense, and Tech Lead — on the same evidence and combines their scores into a final verdict. Running the system 15 times on the same repo produced overall scores ranging from 1.00 to 3.30 on a 5-point scale. Every audit report presents a single number like "Overall score 3.00/5" with no indication of how much that number might change if you ran it again.

My gap is this: when a scoring system that uses LLMs produces different results on the same input across runs, a single reported score is not a measurement — it is one sample from a distribution. But I do not know what statistical tools exist to quantify that distribution, or how to decide how many runs are needed before the reported score is stable enough to trust.

**Specifically: what statistical measures are used to quantify the reliability of a scoring system that produces variable results — such as inter-rater agreement or test-retest reliability — and given an observed range of 1.00 to 3.30 across 15 runs on the same input, how many runs would be needed to report a stable mean score within ±0.5 points at 95% confidence?**

## Artifact Connection

This gap sits in the 15 audit reports in `audit/report_onself_generated/` and in how the final score is reported. Every report states a precise score with no uncertainty bound. Closing this gap would let me add a reliability statement to the audit output — reporting mean ± standard deviation across runs instead of a single point estimate — and document in `README.md` what the observed 2.3-point variance means for how any score should be interpreted by an FDE reading the report.