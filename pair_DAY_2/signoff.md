# Sign-off

**Gap closure status: Closed**

Before reading the explainer, I understood that the system had a model and had tools, but I was using the word "agent" loosely without being able to say what that required technically. I knew something felt off about claiming the model was using tools, but I could not name the mechanism that was missing.

After reading the explainer, I can now say precisely: model-driven tool use requires the model to receive tool schemas, emit a `tool_use` content block, and have its output determine what executes next. TheConversionEngine does none of this. The OpenRouter call in `generation/service.py` sends no `tools` parameter. The model is a Pattern 2 transform and a Pattern 3 gate. The planning lives in Python.

I also understand now why the tau2-bench proactive instruction mattered there but not in the main pipeline: the benchmark gave the model real tool schemas and scored on whether it emitted write-action calls. The pipeline never gives the model schemas at all, so the instruction would have nothing to act on.

The three failure surfaces (orchestrator, model, governance) gave me a concrete mental model for where to look when something goes wrong, which I did not have before.