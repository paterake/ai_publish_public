A chain of coordinating agents landed about 260 commits for me across two major refactors, and the first run needed me only twice: once at the generation cap, once when a director hit its shell-call limit. Every agent commit that went through the commit gate carried an independent PASS bound to the exact staged content. That is hard because coding agents start each session with no memory of the last one, grade their own work too kindly, and are supervised by people who accept far more prompts than they can read.

Context and loop engineering fix much of that inside one session. The joins between sessions stay manual: copying prompts between agents, opening fresh sessions, relaunching the coordinating agent when it hits a session limit. I built a harness so the agents carry those joins.

On 16 November a relay passed on my yes to a fix that would remove the repository copies of the reviewer.

The harness session did not act on it. A decision check had found that the recommendation I approved left out a consequence, since in a cloud session those copies were the only reviewer, so cloud work could no longer commit.

The session put the choice back to me.

Across six days of session logs, the reviewer sent back 56% of changes at least once before passing them. The gate stops commits that skip review and cannot catch forgery. I approved a recommendation that left out a consequence. How do you catch that in yours?

#AI #ClaudeCode #AIGovernance #TechLeadership #SoftwareArchitecture

AI Disclosure: The author originated, researched, and drafted this content from direct implementation experience. AI was used solely to refine prose and formatting.
