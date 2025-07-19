---
layout: post
title: "Stateless LLMs and memory-mutating agents"
date: 2025-07-12 14:00:00 +0000
categories: general
---

I was trying to figure out how large language model (LLM) providers were able to keep so many parallel conversations alive from all the different callers when I realized something that may be obvious to many but I hadn't seen explicitly stated: large language models (LLMs) are stateless. 

An LLM is a pure function that takes a blob of text as input and returns a blob of text as output - no memory or history. Chatbots and agents are built on top of LLMs by accumulating chat messages over time and feeding those into the next LLM request. The sense of history, growth, or continuity is an illusion crafted by the agent, not a inherent property of the system.

This illusion of continuity makes for a useful interface because we can converse with a model in a natural way and refine designs or implementation over a series of exchanges. But I think there are opportunities to improve agents to be had if we're willing to cheat the continuous conversation model.

Some examples of where conversational continuity in agents creates limitations:
- **Outdated file contents**: Agents typically read files as tool calls, and the file contents become part of the message history. Over a longer session though, the original file contents may be edited by the agent, updated by a human making a correction, or may be changed entirely by an auto-formatter for instance. The contents from older file reads are no longer useful, but they are retained in the message history which consumes tokens, confuses the model, and slows responses.
- **Context cruft**: Running build processes (e.g. linting, testing, building) can produce massive outputs which are only briefly useful. For example, an agent renames a widely-used function then run unit tests. Dozens of tests fail, each with a full stack trace. The agent fixes the function name, reruns the tests, and they pass. The agent's context now has the full output of the failed unit tests immortalized in its context, but that build output is no longer useful.
- **Slow information gathering**: As a user of an agent, I start the agent on a new task. It spends the first couple of minutes reading files one by one. Anthropomorphically, it feels like the model is doing the useful work of learning the codebase bit by bit until it understands enough to come up with a plan. But more literally, the model did not need more time or thought but was just missing the context it needed to make its decision. If it had the relevant file contents with the initial prompt, it would have returned the plan immediately. Every extra round trip to the model could also be framed as, "Failed to generate a plan. Try again with more context." 

I've seen a couple common patterns for managing context that I've played with, each with its own limitations:
1. **Codebase scanning tools** like aider's repomap or repomix which help get wider project context into the the model quickly. Aider's repomap scans a large codebase, finds the function definitions, and returns the most important ones (determined by a ranking algorithm). Repomix simply reads an entire codebase and returns the entire contents in a single tool call. These gather a lot of information quickly, but the information does not stay current after the agent makes changes.
2. **Multi-agent flows** where the main agent delegates a task to a subagent which does some work and returns just the summary of what it accomplished. The main agent's context does not get polluted with all the details that the subagent works on. I've used this pattern for builds with some limited success. I prompted a main agent to run a sub-agent to run tests and return only the relevant error message. I found the approach a bit hit or miss, and trying to steer agents that are a layer removed feels like trying to push a button with a long, wobbly stick.
3. **Conversation summarization** when context gets full, where an agent will replace its current message and tool call history with a shorter overview. The approach does clear the context, but in my experience the summaries do not have enough info for the agents to resume their work. Important details like my original prompt, the specific constraints I've given the agent, or the file contents we're editing get lost. Agents need to reread the same files again and often need to the original prompt repeated.


To explore alternative context strategies, I'm building a new proof-of-concept agent. The project's theme is to treat the agent's context - system prompt, conversation history, and tools - as  inputs to be optimized on every request, not static system properties. I'm aiming for the agent to be complete enough to see the ideas function, but don't expect it to be a contender in the increasingly crowded field of CLI agents.

Specific ideas I want to try:
- Live context that includes current file and directory contents with every request as part of the system prompt. Instead of calling tools which return a snapshot in the message log, the agent can add and remove files and directories to always see their current contents. If the agent runs an autoformatter, it will see the formatted contents. If the human user edits a file outside of the agent, the agent will see the updated file contents in the next request, not a stale copy from its last read.
- Pruning the context over time. Have a second agent or LLM-assisted process that removes cruft from the context over time including messages that are no longer needed and tool call results that are aren't relevant. Maybe an LLM 
- Aggressively prefilling the context. Start the agent with the current directory listing and contents of important files like the README. The goal would be to remove the first timid steps agents usually require when first starting to figure out their surroundings.

You can view the project progress at [crowecawcaw/agent](https://github.com/crowecawcaw/agent).