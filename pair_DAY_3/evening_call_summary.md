# Evening Call Summary — Day 3

Samuel confirmed the rule-versus-vocabulary distinction landed clearly — specifically the framing that SimPO's loss goes down either way, making the gradient signal unable to distinguish semantic learning from vocabulary memorization during training. The loss curve analogy was flagged as the part that resolved the confusion: a healthy val loss is necessary but not sufficient evidence of generalization when the val set came from the same template distribution as the training set.

Two pieces of feedback from Samuel:

First, the section covering all four diagnostic tests (loss curve divergence, token-level confidence calibration, out-of-distribution eval, paraphrase eval) was initially read as suggesting all four should be run. The writer clarified that the piece is arguing for paraphrase eval specifically as the highest-signal test for this setup, and the other three are named to show why they are lower signal — not to recommend running them all. The ordering was revised to make the argument structure clearer.

Second, the `methodology.md` paragraph at the end of the signoff was the most immediately actionable part — Samuel drafted the scope statement for the 82% claim during the call itself, using the in-distribution framing from the explainer.

Gap closure confirmed after revision.