---
title: "Learning in public: why I'm starting a build log"
date: 2026-11-04 09:00:00 +0400
description: "A backend engineer's notes on AI-assisted engineering, LLM tooling and systems at scale, written while I'm still figuring it out."
tags: [meta, ai, learning]
---

I've spent my career on backend systems: 5G test software at Nokia, high-traffic merch stores for global artists, and now the property platform at Dubizzle. Most of what I learned along the way lives in pull requests, Slack threads and my own head. None of those are easy for anyone else to read.

The last year changed how I work. AI tools moved from autocomplete to something closer to a junior teammate. I started building agent pipelines, wiring tools together over MCP, and reading about RAG and vector databases. I learn by building, and I've noticed I only really understand something once I can explain it.

So this is the explanation part.

## What I'll write about

- **AI-assisted engineering in practice.** Multi-agent coding pipelines, guideline files, what goes wrong and how I fix it. Less hype, more config and debugging.
- **LLM tooling.** Model Context Protocol (MCP), retrieval-augmented generation, vector databases, and how LLMs handle memory and context.
- **Backend fundamentals.** Distributed-systems ideas like the CAP theorem, async pipelines, idempotency, and the trade-offs that come up running a marketplace at scale.
- **Side projects.** The things I build on weekends, like [a multi-tenant wedding planner](https://planner-six-inky.vercel.app) and [a budget tracker](https://expenses-kappa-one.vercel.app), and what they taught me.

## How I'll write it

Each post should be something I would have wanted to find while I was stuck:

1. **The problem**, stated plainly.
2. **What I tried**, including the dead ends.
3. **What worked**, with real config or code where I can share it.
4. **What I'd do differently**.

Work posts will stick to patterns and lessons and leave out anything internal. If something here is wrong, I'd rather hear about it. That's half the point of writing in public.

First up: [how I split one AI coding assistant into five specialised agents](/blog/multi-agent-claude-code-pipeline/).
