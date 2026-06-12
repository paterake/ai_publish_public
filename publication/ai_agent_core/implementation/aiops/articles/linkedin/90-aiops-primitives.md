The signals the agent built to catch pipeline failures eventually caught failures in the agent's own code.

Code review had passed all three custom implementations. The signals found problems in each: scores that weren't comparable across runs, and circuit state transitions missing from telemetry.

Two modules the agent had called done were not. The classifier was calling the LLM on nearly every query when cheaper layers could have answered most of them. The visualisation pipeline was spending its token budget on chain-of-thought before producing any charts. Both had passed code review. Neither was visible from the output alone.

This is the scale problem. A coding agent generates more code than one reviewer can check. A pipeline processing years of operational data has no path for turn-by-turn human verification. The signals have to be the review mechanism.

Fourteen primitives. Zero required cloud dependencies. The operational layer wasn't designed. Every failure that had no signal forced the agent to build one.

Could the signals in your pipeline find problems in the pipeline itself?

#AI #ClaudeCode #LLMOps #VibeCoding #AIGovernance
