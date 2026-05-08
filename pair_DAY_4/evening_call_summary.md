# Evening Call Summary — Day 4

Martha confirmed the covariance cancellation formula landed clearly — specifically the framing that `Var(paired) = Var(A) + Var(B) - 2·Cov(A,B)` versus `Var(unpaired) = Var(A) + Var(B)`, and that the entire benefit of pairing lives in the subtracted term. The simulation showing 85% CI width reduction at task-level correlation of 0.97 was flagged as the most convincing part — it made the abstract formula concrete with numbers matching her actual n=59 setup.

Two pieces of feedback from Martha:

First, the section on "when pairing can hurt" was initially unclear — she read it as a warning that pairing might be wrong for her setup. The writer clarified that high positive covariance is the expected case when both models are evaluated on the same task distribution, which is always true for her benchmark. The section was reworded to make clear it is a general caveat, not a concern specific to her run.

Second, the `methodology_rationale.md` paragraph at the end was the most immediately useful part — she pasted it directly into her repo during the call, replacing the implicit assumption that paired bootstrap was the right choice with a defended one-paragraph explanation.

Gap closure confirmed after revision.