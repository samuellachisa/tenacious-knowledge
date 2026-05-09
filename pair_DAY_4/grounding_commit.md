# Grounding Commit — Day 4

## What Changed

**File edited:** `README.md` and `audit/report_onself_generated/` reporting logic in `automation-auditor`

**Specific edit:** The score reporting section of the audit output was revised to report mean ± CI across runs instead of a single point estimate, and the README was updated to document the observed variance and what it means for interpreting any reported score.

## Before (README.md)

```
## Scoring

The Automaton Auditor produces an overall score from 0–5 based on three
parallel LLM judges: Prosecutor, Defense, and Tech Lead. Scores reflect
the quality of code, documentation, and engineering decisions in the
audited repository.
```

## After (README.md)

```
## Scoring

The Automaton Auditor produces an overall score from 0–5 based on three
parallel LLM judges: Prosecutor, Defense, and Tech Lead. Each judge is an
independent stochastic draw at the model's default temperature. The Chief
Justice combines the three scores through a deterministic averaging formula,
but stochastic variance is introduced before that step and cannot be fully
eliminated downstream.

Observed variance across 15 runs on the same repository: range 1.00–3.30,
SD ≈ 0.61. A single reported score should be interpreted with a ±0.5
uncertainty at 95% confidence. For a stable mean score within ±0.5 at 95%
confidence, run the auditor at least 9 times on the same target and report
the mean and CI. The 15-run dataset already collected yields a 95% CI
half-width of ±0.34.
```

## Before (audit output)

```
Overall score: 3.00 / 5
```

## After (audit output, when run in multi-run mode)

```
Overall score: 2.84 / 5  (mean across 15 runs, 95% CI: 2.50 – 3.18)
Single-run range observed: 1.00 – 3.30
Minimum runs for ±0.5 stability: 9
```

## Why This Edit

A single reported score from an LLM-judge system with temperature > 0 is not a measurement — it is one sample from a distribution. The previous output format implied precision that the system does not have. Reporting the mean and CI alongside the single-run range gives any Forward-Deployed Engineer reading the report an accurate account of what the number means and how much weight to put on it. The minimum-runs figure (9) makes the reliability threshold concrete and actionable rather than a vague caveat.