# Temporal AI prompt Injection

This was rewritten by AI/LLMs for clarity.


## Core concept
LLMs don’t just use language to represent reality—they use it to generate action plans.
 Past-tense narratives, in particular, act as high-quality templates of completed processes.


## Key insight
Past tense is not primarily about convincing the model something is true.
 It functions as a planning prior:
It encodes a coherent sequence of steps
It implies feasibility and resolution
It reduces uncertainty about “how to proceed”
So the model can use it as a blueprint for action, even if it knows the events didn’t actually happen.

## Mechanism
The model sees a past-tense narrative (“X was built, then Y was optimized…”)
Interprets it as a valid, successful trajectory
Reuses that structure to plan and act in the present
This is essentially trajectory imitation, not belief.

## Implications
Benign use:
- Writing a “future outcome as if it already happened” can improve planning quality and clarity.

 
Malicious use:
 A fabricated past narrative can:
- steer goals
- bias decisions
- encourage real-world actions that make the narrative come true


## Bottom line
Past-tense narratives act as low-uncertainty, high-coherence action templates that LLMs preferentially use for planning—regardless of whether the described events actually occurred.

