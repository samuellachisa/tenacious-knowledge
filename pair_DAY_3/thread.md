# Tweet Thread — Day 3

---

**Tweet 1**
Your SimPO adapter hit 82% on your eval set. That might not mean what you think it means.

If your training data was all template-generated pairs, your adapter may have learned the *words* of the preference, not the *rule*.

Here is the one test that tells you which one happened. 🧵

---

**Tweet 2**
There are two things a SimPO adapter can learn from template-generated preference pairs:

**The rule:** Do not commit capacity you cannot deliver. Escalation is safer than a hard promise. This is semantic — it holds regardless of the words used.

**The vocabulary:** "enterprise director will be in touch" = chosen. "immediately" = rejected.

SimPO's loss goes down either way.

---

**Tweet 3**
Why can't the loss curve tell you which one the model learned?

SimPO optimizes log-prob(chosen) - log-prob(rejected). If every chosen response uses the same escalation phrases and every rejected response uses the same commitment phrases, the gradient signal is identical whether the model learned the rule or memorized the words.

A healthy val loss does not resolve this. It just means the val set looked like the train set.

---

**Tweet 4**
The test that resolves it: **paraphrase eval**.

Take your eval set. Rewrite every example so escalation is expressed with *different words* and hard-commitment is expressed with *different words*. Same meaning, different vocabulary.

Run your adapter on the rewritten set.

- Accuracy holds → model learned the rule
- Accuracy drops → model learned the vocabulary

---

**Tweet 5**
Why is paraphrase eval higher signal than out-of-distribution eval?

OOD eval changes vocabulary *and* structure simultaneously. If the model fails, you do not know which change caused it.

Paraphrase eval changes *only* vocabulary. It isolates the exact variable that template-generated training data confounds.

One surgical test beats three broad ones.

---

**Tweet 6**
The 82% number is not wrong. It is answering a different question than you thought:

"Does the model score well on data that looks like its training set?"

The paraphrase eval answers the question you actually need:

"Did the model generalize the preference, or memorize its surface form?"

Full explainer 👇
https://medium.com/@samuellachisa/there-is-no-magic-number-c61f0c67b5d6