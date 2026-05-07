# tenacious-knowledge

![Week](https://img.shields.io/badge/week-12-blue)
![Days](https://img.shields.io/badge/days-5-brightgreen)
![Blog Posts](https://img.shields.io/badge/blog%20posts-5-orange)
![Tweet Threads](https://img.shields.io/badge/tweet%20threads-5-1DA1F2)
![Gaps Closed](https://img.shields.io/badge/gaps%20closed-10-purple)
![License](https://img.shields.io/badge/license-CC%20BY%204.0-green)

Week 12 research and pedagogy work built on top of the Tenacious Conversion Engine — a B2B sales AI agent and benchmark system shipped in Weeks 10 and 11.

Weeks 10 and 11 were about building and shipping. Week 12 is about understanding what was built deeply enough to teach it. The deliverable is not a new system — it is the ability to explain every engineering choice made in the prior two weeks, defend it under pressure, and produce public writing that closes the same gaps for other engineers.

---

## How It Works

Each day follows a five-act loop:

**Morning** — I explore the day's topic, surface gaps in my own work, and triage them to find the one whose closure would meaningfully change how I think about what I built. I draft a precise question grounded in a specific artifact from my portfolio.

**Morning call** — My partner and I interrogate each other's draft questions in a live call. We push back, rewrite, and commit to a final question each. The sharpening is the work.

**Day** — I receive my partner's final question. I read at least two canonical papers, run at least one tool hands-on, and write a 600–1000 word blog post and tweet thread that closes their gap.

**Evening call** — We exchange feedback on each other's explainers. The writer revises. The asker signs off on whether the gap actually closed.

**Commit** — The asker makes a concrete edit to their Week 10/11 portfolio work, grounded in what they just learned. The explainer ships publicly.

---

## Daily Submissions

Each day folder contains eight files:

| File | Contents |
|------|----------|
| `question.md` | My final sharpened question with the named portfolio artifact it connects to |
| `morning_call_summary.md` | What was ambiguous in the original draft and how the question was sharpened |
| `explainer.md` | The 600–1000 word blog post I wrote for my partner's question |
| `thread.md` | The tweet thread or Medium post compressing the explainer |
| `evening_call_summary.md` | Feedback exchanged and what was revised |
| `signoff.md` | My partner's gap-closure verdict and what they now understand |
| `grounding_commit.md` | The actual edit made to my Week 10/11 portfolio with explanation |
| `sources.md` | Two canonical papers and the tool used hands-on |

---

## Public Artifacts

| Day | Topic | Blog Post | Thread |
|-----|-------|-----------|--------|
| 1 | Agent and Tool-Use Internals | [![Blog](https://img.shields.io/badge/blog-post-orange)](https://medium.com/@samuellachisa/who-is-actually-using-the-tools) | [![Thread](https://img.shields.io/badge/medium-thread-1DA1F2)](https://medium.com/@samuellachisa/who-is-actually-using-the-tools) |
| 2 | Agent and Tool-Use Internals | [![Blog](https://img.shields.io/badge/blog-post-orange)]() | [![Thread](https://img.shields.io/badge/medium-thread-1DA1F2)]() |
| 3 | Training and Post-Training Mechanics | [![Blog](https://img.shields.io/badge/blog-post-orange)]() | [![Thread](https://img.shields.io/badge/medium-thread-1DA1F2)]() |
| 4 | | [![Blog](https://img.shields.io/badge/blog-post-orange)]() | [![Thread](https://img.shields.io/badge/medium-thread-1DA1F2)]() |
| 5 | | [![Blog](https://img.shields.io/badge/blog-post-orange)]() | [![Thread](https://img.shields.io/badge/medium-thread-1DA1F2)]() |

---

## End of Week Deliverables

| File | Contents |
|------|----------|
| [`synthesis.md`](synthesis.md) | 1500-word week synthesis — ten gaps closed, most surprising finding, canonical reading list |
| [`canonical_list.md`](canonical_list.md) | Annotated papers, tools, and patterns worth every Forward-Deployed Engineer reading |
| [`portfolio_update.md`](portfolio_update.md) | One-page summary of how the five grounding commits improve the Week 10/11 portfolio, written for a hiring manager |

---

## Grounding Commits

Every gap closed this week produces a concrete edit to the Week 10/11 portfolio. Links to commits below as they are made.

| Day | File Edited | What Changed | Commit |
|-----|-------------|--------------|--------|
| 1 | `README.md`, `method.md` | Architecture description revised: orchestrator-controlled pipeline, model roles clarified as rewriter and governance gate, proactive tool-use instruction scoped to tau2-bench context only | |
| 2 | `orchestration/handoff.py`, `method.md` | Documented intent classification gap in `route_inbound_message()`; added `classify_reply_intent` design spec to `method.md` | |
| 3 | `methodology.md` | 82% capacity_honesty claim bounded to in-distribution template eval; paraphrase robustness check planned for v2; Pareto-optimality argument scoped to template distribution | |
| 4 | | | |
| 5 | | | |

---

## Portfolio

The Week 10 and 11 work this week builds on:

[![tenacious_bench](https://img.shields.io/badge/GitHub-tenacious__bench-181717?logo=github)](https://github.com/samuellachisa/tenacious_bench)
[![HuggingFace Dataset](https://img.shields.io/badge/🤗%20dataset-tenacious--bench-blue)](https://huggingface.co/datasets/samuellachisa/tenacious-bench)
[![HuggingFace Model](https://img.shields.io/badge/🤗%20model-simpo--lora-orange)](https://huggingface.co/samuellachisa/tenacious-bench-simpo-lora)