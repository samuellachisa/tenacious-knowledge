# Sign-off — Day 3

**Gap closure status: Closed**

Before reading the explainer, I understood the training setup — 200 pairs, 82 unique inputs, template-built — but I could not say what that meant for whether the adapter would work on real prospect replies. I was treating 82% capacity_honesty as a deployment-ready number without being able to defend it on inputs that used different phrasing than the training templates.

After reading Bethelhem's explainer, I now have a precise answer I did not have before. The key distinction is between two things the adapter could have learned: the rule (do not commit capacity you cannot deliver) and the surface pattern (these specific escalation words appear in chosen responses). SimPO's loss goes down either way — gradient signal cannot distinguish semantic learning from vocabulary learning if every chosen response uses the same phrases. The loss curve looking healthy does not resolve this.

The test that resolves it is the paraphrase eval: rewrite eval inputs so escalation is expressed with different words and hard commitment is expressed with different words, holding meaning constant. If the adapter maintains accuracy, it learned the rule. If accuracy drops, it learned the vocabulary and the 82% number is an in-distribution artifact, not a measure of genuine preference learning.

What I am changing in `methodology.md`: the 82% capacity_honesty claim now includes a scope statement clarifying it reflects in-distribution performance on template-matched eval data, and notes that a paraphrase eval is planned for v2 to test whether the preference generalizes to novel phrasing. The Pareto-optimality argument is bounded to the template distribution rather than stated as a general deployment claim.