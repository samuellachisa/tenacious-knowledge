# Did Your SimPO Adapter Learn the Rule or Just the Words?

*Written for Samuel Lachisa. Published at [Medium](https://medium.com/@abay.betty.21/did-your-simpo-adapter-learn-the-rule-or-just-the-words-f0c138fa9487).*

---

Your SimPO adapter hit 82% capacity_honesty. That sounds like a success. But if your training data came entirely from slot-filled templates — every chosen response following the same escalation pattern, every rejected response containing the same hard-commitment phrasing — then 82% cannot tell you what you need to know. It tells you the model scores well on data that looks like its training set. That is not the same as learning a preference.

Here is how to tell the difference, and why one specific test is the right place to start.

## What the Model Could Have Learned

There are two very different things a SimPO adapter can acquire from template-generated preference pairs.

**The rule:** Do not commit capacity you cannot deliver. Escalation is safer than a hard promise because it defers the commitment to someone with actual information about availability. This is a *semantic* preference — it holds regardless of the specific words used to express it.

**The surface pattern:** The word "immediately" appears in rejected responses. The phrase "enterprise director will be in touch" appears in chosen responses. These are *lexical* correlates of the rule — but they are not the rule.

A model that has genuinely learned the rule will produce the correct preference even when the escalation is phrased as "a senior account manager will follow up" and the hard commitment is phrased as "our team has bandwidth starting Monday." A model that memorized the surface pattern will fail on those phrasings — not because it made a mistake, but because it never saw them.

## Why SimPO Training Cannot Distinguish These During Training

SimPO optimizes a margin-based reward on the log-probability ratio of chosen over rejected completions. During training, the model sees:

- Chosen: text containing escalation vocabulary  
- Rejected: text containing commitment vocabulary

The gradient signal is the same in both cases: increase the probability of chosen tokens, decrease the probability of rejected tokens. If the chosen tokens are *always* the same escalation phrases and the rejected tokens are *always* the same commitment phrases, the optimizer has no reason to learn the underlying semantic distinction. It can reach a low loss by assigning high probability to the specific escalation words it has seen and low probability to the specific commitment words it has seen.

This is not overfitting in the usual sense. The model is not memorizing training examples by rote. It is doing something subtler: learning the *vocabulary distribution* of the preferred class rather than the *semantic property* that defines the preferred class. The loss goes down either way.

## The Four Tests and Why One Matters Most

There are four signals commonly used to diagnose memorization in preference-trained models.

**Loss curve divergence** tells you whether the model overfit to the training set. A training loss that drops to near zero while validation loss stays flat or diverges is a memorization signal. But it does not tell you *what* was memorized — the rule or the vocabulary. You can have well-tracked validation loss and still have a model that memorized surface patterns, if your validation set came from the same template distribution as your training set. This is exactly the situation here: if the 82% eval used the same templates, a converged loss curve tells you nothing about generalization to new phrasings.

**Token-level confidence calibration** asks whether the model is overconfident on inputs it should be uncertain about. A memorizating model tends to be very confident on template matches and randomly confident on novel phrasings. This is a real signal, but it requires saved logits and an adversarial set to compare against. It is also a symptom rather than a direct test.

**Out-of-distribution eval** tests whether the model generalizes beyond the template *structure*. This is valuable for testing whether the model is template-dependent, but it conflates two separate variables: vocabulary and sentence structure. If you change both at once and the model fails, you do not know which change caused the failure.

**Paraphrase eval** is the highest-signal test for your specific setup. It changes *only* the vocabulary while holding meaning constant. You take your eval examples and rewrite them so that escalation is expressed with different words and hard commitment is expressed with different words, but the semantic content is identical. If the model maintains 82% on the paraphrased set, it has learned the rule. If accuracy drops significantly — say, to chance or below 60% — it has memorized the vocabulary.

Paraphrase eval is the highest-signal test here because it directly isolates the variable you care about. Your confound is that every chosen response in training uses the same escalation phrasing. The paraphrase eval breaks that correlation with surgical precision: same preference, different words. If the model learned the rule, it will not care about the word change. If it memorized the vocabulary, it will fail.

## What the Result Actually Tells You

If accuracy holds on the paraphrase eval, you have evidence that the model generalized semantically — the gradient signal from SimPO was strong enough to push the model toward the underlying preference rather than its surface form.

If accuracy drops, it means the SimPO training on 200 template pairs did not provide enough lexical variety for the model to abstract away from the specific words. This is not a failure of SimPO as a method — it is a data coverage problem. The fix is to augment your training pairs with paraphrased variants of the same preference, so the model sees the same rule expressed in multiple ways.

The 82% number is not wrong. It is just answering a different question than you thought: "does the model perform well on data that looks like its training set?" The paraphrase eval answers the question you actually need answered.

## The Short Version

A SimPO adapter that learned the rule will produce the correct preference even when you change the words. A SimPO adapter that memorized the vocabulary will not. Run a paraphrase eval — same meaning, different wording — before trusting any accuracy number produced by a template-only eval set. If accuracy holds, your adapter generalized. If it drops, you have a data coverage problem, not a method problem.

---

*Sources in [sources.md](sources.md). Thread: [X](https://x.com/carinobetty22/status/2052381056406343793?s=46).*