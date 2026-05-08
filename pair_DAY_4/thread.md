# Tweet Thread — Day 4

---

**Tweet 1**
A paired bootstrap CI can be 85% narrower than an unpaired one on the same data.

My partner built a sales agent benchmark with a 3pp performance target. Without pairing, that target is statistically invisible at n=59 tasks. With pairing, it is detectable.

Here is the one formula that explains why. 🧵

---

**Tweet 2**
When you estimate delta = detection_rate(ORPO) − detection_rate(Claude), the variance depends on how you resample:

```
Unpaired: Var(delta) = Var(A) + Var(B)
Paired:   Var(delta) = Var(A) + Var(B) − 2·Cov(A,B)
```

Pairing subtracts 2·Cov(A,B) from the variance.

That one term is the entire mechanism.

---

**Tweet 3**
Why is Cov(A,B) large in agent benchmarks?

Both models are scored on the same tasks. Hard tasks — ambiguous signals, borderline data — make both models more likely to fail. Easy tasks make both more likely to pass.

This shared difficulty creates high positive covariance. Pairing preserves it. Unpaired bootstrap breaks it and treats it as noise.

---

**Tweet 4**
At task-level correlation of 0.97 (realistic for two judges on the same benchmark):

```
Unpaired CI: [−0.15, +0.19]  width = 0.34
Paired CI:   [+0.00, +0.05]  width = 0.05
```

The unpaired CI includes zero — cannot confirm ORPO is better at all.
The paired CI excludes zero — 3pp target detectable.

Same data. One `zip()` call difference.

---

**Tweet 5**
The paired bootstrap and the paired t-test exploit the same 2·Cov(A,B) cancellation.

The bootstrap is preferred for detection rates because binary outcomes violate the normality assumption that the t-test requires.

Same mathematical insight. Different distributional assumption.

---

**Tweet 6**
If you are reporting a benchmark delta without pairing, your CI is wider than it needs to be — and small but real improvements are invisible.

Full explainer with derivation, simulation code, and what to write in your methodology doc 👇

https://medium.com/@samuellachisa/why-pairing-is-the-whole-point-the-mathematics-of-paired-bootstrap-in-agent-benchmarks-7c079e1572c0?postPublishedType=initial