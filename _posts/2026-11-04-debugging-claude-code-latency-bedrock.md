---
title: "When the AI hangs: debugging Claude Code latency on Amazon Bedrock"
date: 2026-11-04 11:00:00 +0400
description: "Claude Code running through Bedrock started hanging and responding slowly. Here's how I separated model latency from context bloat, and what fixed it."
tags: [ai, claude-code, aws, debugging]
series: AI-assisted engineering
---

Our Claude Code setup runs against **Amazon Bedrock** in `eu-west-1` rather than Anthropic's API directly, alongside the AWS infrastructure we already run on EKS. It worked well until it didn't: sessions started **hanging**, and responses that used to take seconds took much longer.

"The AI is slow" isn't something you can fix. I needed to know *which part* was slow.

## Step 1: get the mental model right

The first confusion was conceptual. **Claude Code and the Anthropic Console are different things.** The Console is Anthropic's web platform for API keys, usage and prompt testing. Claude Code is the agentic CLI. When Claude Code is pointed at Bedrock, the Console is out of the loop entirely: auth, quotas, model availability and latency all come from your AWS account and region.

The Bedrock setup comes down to a few environment variables:

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=eu-west-1
export ANTHROPIC_MODEL=<bedrock model or inference profile id>
```

So any debugging had to happen on the AWS side, not in a Console dashboard.

## Step 2: take the tool out of the equation

There were two possible culprits:

1. **The endpoint**: Bedrock itself being slow or throttling in that region.
2. **The payload**: Claude Code sending so much context that every turn was slow to process.

To separate them, I called Bedrock **directly** with a tiny prompt and timed it:

```bash
time aws bedrock-runtime converse \
  --region eu-west-1 \
  --model-id "$ANTHROPIC_MODEL" \
  --messages '[{"role":"user","content":[{"text":"Reply with OK."}]}]'
```

This gives a baseline: how fast the model answers when nothing else is involved. If the direct call is also slow, the problem is upstream (region, quota, model choice). If the direct call is fast but Claude Code is slow, the problem is what the session is sending.

## Step 3: fix the context, not the model

The symptoms pointed at **context bloat**. An agentic session accumulates everything: files it read, command output, long test logs, earlier attempts. Every new turn sends that history again, so turns get slower and the model gets worse at focusing on the actual task.

What helped:

- **Compact early.** `/compact` summarises the conversation so far and continues from the summary. I now run it at natural breakpoints, for example after a feature is implemented and before review starts, instead of waiting for things to slow down.
- **Keep heavy directories out of reach.** Generated files, vendored packages, static builds and large fixtures should never end up in context. In Claude Code this goes in permission deny rules in `.claude/settings.json`:

  ```json
  {
    "permissions": {
      "deny": [
        "Read(./node_modules/**)",
        "Read(./staticfiles/**)",
        "Read(./**/fixtures/*.json)"
      ]
    }
  }
  ```

- **Match the model to the job.** Not every step needs the largest model. Switching with the `--model` flag (or `ANTHROPIC_MODEL`) for lighter tasks like linting or summarising is a cheap way to cut wait times. Save the largest model for the steps that need deep reasoning.

## What I took away

- **Measure the layers separately.** A direct API call is the cheapest experiment you can run, and it rules out half the problem space in a minute.
- **Context is a resource, so budget it.** This is also why splitting work across [specialised agents](/blog/multi-agent-claude-code-pipeline/) helps: each one carries only what it needs.
- **Know which platform you're on.** Many "it's broken" moments with AI tooling are really "I'm looking at the wrong dashboard" moments.
