# Sources

## Canonical Sources

**1. Hong et al., "ORPO: Monolithic Preference Optimization without Reference Model" (2024)**
https://arxiv.org/pdf/2403.07691

The primary paper defining the ORPO objective. Source for the full loss formulation used in the explainer:

```
L_ORPO = L_SFT + λ · L_OR

L_SFT = -(1/M) Σ log P_θ(y_w | x)
L_OR  = -log σ( log( odds_θ(y_w|x) / odds_θ(y_l|x) ) )
odds_θ(y|x) = P_θ(y|x) / (1 - P_θ(y|x))
```

Used to explain why the raw ORPO loss value is not directly comparable to cross-entropy loss, and why a falling eval loss of 0.3851 means both the SFT signal and the preference signal are generalizing simultaneously. Also the source for `orpo_beta = λ = 0.1` as the weighting term between the two loss components, which maps directly to the `orpo_beta` hyperparameter in Betty's `training/training_run.log`.

---

**2. Brownlee, "How to use Learning Curves to Diagnose Machine Learning Model Performance," MachineLearningMastery.com**
https://machinelearningmastery.com/learning-curves-for-diagnosing-machine-learning-model-performance/

The authoritative practical reference for reading train-val loss trajectories. Source for the three-regime framework (healthy generalization, overfitting, underfitting) and the definition of the generalization gap as the expected gap between training and validation loss learning curves. Directly supports the core argument that no fixed threshold for G(t) can define overfitting — what matters is the direction of `dL_eval/dt`, not the absolute value of the gap.

---

## Tool or Pattern Used

**Direct reading of `training/training_run.log` in bettyabay/tenacious-bench**
`git clone https://github.com/bettyabay/tenacious-bench.git`

Files read:
- `training/training_run.log` — full step-by-step training table including train loss, eval loss, rewards/accuracy, and rewards/margins at every 20-step interval from step 0 to step 200
- `training/config.yaml` — confirmed `orpo_beta = 0.1`, `lora_r = 16`, `lora_alpha = 16`, `max_steps = 200`

The generalization gap table in the explainer was computed directly from the log:

```python
# Steps where eval loss is recorded
steps =    [20,     40,     60,     80,     100,    120,    140,    160,    180,    200]
train_l =  [2.1163, 0.9471, 0.5534, 0.3812, 0.2734, 0.2103, 0.1812, 0.1634, 0.1501, 0.1411]
eval_l =   [2.4905, 1.1823, 0.8012, 0.6534, 0.5501, 0.4923, 0.4512, 0.4201, 0.3952, 0.3851]
gap =      [round(e - t, 4) for t, e in zip(train_l, eval_l)]
# [0.3742, 0.2352, 0.2478, 0.2722, 0.2767, 0.2820, 0.2700, 0.2567, 0.2451, 0.2440]
```

The gap peaked at step 120 (+0.2820) and narrowed to +0.2440 by step 200. The overfitting condition `dL_eval/dt > 0` was never satisfied across all 200 steps. These are verifiable claims — anyone can open the log and recompute.

---

## Follow-On Reading

- Gao et al., "Scaling Laws for Reward Model Overoptimization" (2022), arXiv:2210.10760 — the complementary failure mode: what happens when reward margin grows too large and the adapter optimizes against the preference signal rather than generalizing it. The other side of the coin from underfitting.
- Hugging Face `EarlyStoppingCallback` documentation — https://huggingface.co/docs/transformers/main_classes/callback#transformers.EarlyStoppingCallback — the practical implementation of the early stopping criterion described in the explainer, using `metric_for_best_model="eval_loss"` and `greater_is_better=False`