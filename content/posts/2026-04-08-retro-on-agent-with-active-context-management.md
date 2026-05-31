---
title: "Retro on agent with active context management"
date: 2026-04-08T14:00:00+00:00
---

I created a coding agent a year ago to experiment with the idea of active context management. At the time, agents had limited context that filled quickly and cost increasing amounts of money to maintain. My main idea was letting the agent prune and compact its context over time to keep just the necessary info around. In theory, with a leaner focused context, we'd pay less for input tokens and the model could maintain focus for longer periods of time.

A couple things changed in the past year making active context management unnecessary or harmful:

1. **Prompt caching changes the economics of agents.** If an LLM is invoked with an input whose beginning exactly matches a previous invoke, the LLM provider can reuse a substantial amount of computation from the earlier prompt. Cache hits reduce costs by 90% or more. If the prompt changes between invokes (for example, by compacting earlier messages), we get an expensive cache miss. Now, costs are better optimized by keeping the conversation consistent between invokes, not by shrinking it. [This post](https://x.com/trq212/status/2024574133011673516) dives into how critical prompt cache optimization is for Claude Code economics.

2. **Models defend against filling context quickly.** Where previously a verbose build output might fill an agent's context in a single tool call, current models use tricks like looking at just the final lines of the build output to find error messages while ignoring the bulk of the output.
