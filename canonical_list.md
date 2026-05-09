# Canonical Reading List

Papers, tools, and patterns worth every Forward-Deployed Engineer reading. Annotated for what each one actually changes about how you build and evaluate LLM systems.

---

## Papers

**Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022)**
arXiv:2210.03629

Read this before calling anything an agent. ReAct formalizes the plan-act-observe loop: the model receives tool schemas, emits tool call syntax, and its output determines what executes next. Most production pipelines do not implement this loop at all — Python sequences the calls and the model is a text transformer inside a fixed slot. The paper makes the distinction concrete and gives you the vocabulary to describe what you actually built.

---

**Hong et al., "ORPO: Monolithic Preference Optimization without Reference Model" (2024)**
arXiv:2403.07691

The primary source for the ORPO loss formulation. Explains why ORPO eval loss is not directly comparable to cross-entropy loss (it combines an SFT term and an odds-ratio preference term), what the `orpo_beta` hyperparameter actually weights, and what a healthy train-eval curve looks like for a joint objective. Essential reading before reporting any ORPO training result.

---

**Azar et al., "A General Theoretical Paradigm to Understand Learning from Human Feedback" (2023)**
arXiv:2310.12036

The theoretical grounding for SimPO and other reference-free preference objectives. Explains the margin-based loss, why the SimPO gradient signal is the same whether the model learned a behavioral rule or memorized surface vocabulary, and what data diversity requirements follow from that. The conceptual foundation for the paraphrase eval argument.

---

**Dror et al., "Deep Dominance — How to Properly Compare Deep Neural Models" (ACL 2019)**
arXiv:1811.01808

The paper that makes the paired bootstrap non-optional for benchmark comparisons. Three arguments that are directly applicable to any agent evaluation: (1) pairing is required whenever both models are scored on the same task set; (2) binary detection rates violate the normality assumption that parametric tests require, making the bootstrap the correct tool; (3) task-level variance in NLP benchmarks is high enough that unpaired tests are severely underpowered for small effect sizes. A 3pp improvement is statistically invisible without pairing at n=59 tasks.

---

**Efron and Tibshirani, "An Introduction to the Bootstrap" (1993)**
Chapman & Hall/CRC. ISBN: 978-0412042317

The derivation of paired bootstrap variance reduction lives in Chapter 9: `Var(paired) = Var(A) + Var(B) - 2·Cov(A,B)`. Chapter 14 connects bootstrap CIs to hypothesis testing. The mathematical foundation behind every CI width claim in Week 12.

---

**Koo and Mae, "A Guideline of Selecting and Reporting Intraclass Correlation Coefficients for Reliability Research"**
Journal of Chiropractic Medicine, 15(2), 155–163. (2016)

The practical reference for ICC threshold interpretation: below 0.50 is poor, 0.50–0.75 is moderate, 0.75–0.90 is good, above 0.90 is excellent. Includes a decision table for selecting the right ICC variant (one-way vs two-way, absolute agreement vs consistency) depending on whether raters are fixed or random. Required reading before reporting any multi-judge scoring result.

---

**Zheng et al., "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (NeurIPS 2023)**
arXiv:2306.05685

Section 4 is the empirical basis for treating LLM judge variance as a measurement problem rather than an implementation quirk. Documents position bias, verbosity bias, and self-enhancement bias as systematic sources of judge inconsistency. Establishes that multi-judge panels with averaging reduce but do not eliminate variance, and that temperature is a first-class design decision for any evaluation system using LLM judges.

---

**Shrout and Fleiss, "Intraclass Correlations: Uses in Assessing Rater Reliability" (1979)**
Psychological Bulletin, 86(2), 420–428.

The original paper defining the ICC family. The 1979 notation (ICC(1,1), ICC(2,1), ICC(3,1)) is still the standard. Necessary for understanding why different ICC variants give different numbers on the same data and which one applies to a multi-judge system where the judges are fixed (always the same three LLMs) versus random (any judge from a population).

---

## Tools

**Langfuse** — langfuse.com

The instrumentation layer for LLM pipelines. Three concepts to internalize before using it: a trace groups all calls for one user request; a span is one step inside a trace; a generation is an LLM call with input, output, token counts, and cost. The value is not the dashboard — it is the ability to correlate a bad output with the exact prompt, model, and pipeline stage that produced it. Install `langfuse`, initialize with `LANGFUSE_PUBLIC_KEY` and `LANGFUSE_SECRET_KEY`, wrap each model call in `trace.generation()`. Three function calls to go from flat logs to queryable traces.

**Hugging Face `trl` library — SimPO and ORPO trainers**
github.com/huggingface/trl

The reference implementation for preference training objectives. `SimPOConfig` and `ORPOConfig` expose every hyperparameter discussed in the papers with documented defaults. Running a training job with this library and reading the source for `SimPOTrainer.compute_loss` is the fastest path from paper to mechanical understanding.

---

## Patterns

**Paraphrase eval for preference adapters.** After training a SimPO or ORPO adapter on template-generated preference pairs, run a second eval set where the same semantic preferences are expressed with different vocabulary. Same meaning, different words. If accuracy holds, the adapter learned the behavioral rule. If it drops, it learned the template vocabulary. This test is cheap — the eval set is a one-time rewrite of the existing holdout — and it is the only test that isolates vocabulary memorization from semantic learning.

**Mean ± CI reporting for LLM judge scores.** Any scoring system that uses LLM calls at temperature > 0 produces a distribution, not a point. Run the system at least 9 times on each target and report `mean ± half-width of 95% CI` rather than a single score. With SD around 0.6 on a 5-point scale, a single-run score has a true uncertainty of roughly ±0.5 at 95% confidence — wide enough to flip most pass/fail thresholds.

**Three-pattern framework for documenting LLM systems.** Before writing "agent" in any README, identify which pattern the model is actually playing: (1) model-as-planner, where the model receives tool schemas and its output determines what executes; (2) orchestrator-as-planner, model-as-transform, where Python sequences the calls and the model rewrites content in a fixed slot; (3) model-as-reviewer, where the model returns a typed verdict that gates one downstream branch. Most production systems are Pattern 2 or 3. Document the pattern accurately — it determines where failures live and how they should be debugged.