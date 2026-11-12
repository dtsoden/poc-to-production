---
name: guardrails
description: Set production guardrails for a new project before any code is written. Writes the build rules from the poc-to-production standard into the repository's agent instructions file so every session follows them. Use at the start of a new app or proof of concept.
argument-hint: "[what you're building]"
disable-model-invocation: true
---

Read `${CLAUDE_SKILL_DIR}/../../prompts/prompt-0-guardrails.md` and follow the prompt inside its code block as if the user had pasted it after their first prompt.

What the user is building: $ARGUMENTS

If that is empty, ask the user to describe the project in a few sentences first, then continue.

In this first session:

1. Write the rules from the prompt into the repository's agent instructions file. Use CLAUDE.md if the repository has one or has no instructions file yet; use AGENTS.md or CONTRIBUTING.md if that is what the repository already uses. Keep the rules as written. Add one line saying the full measurement is the standard at https://github.com/dtsoden/poc-to-production.
2. Ask the user whether they also want the measurement: if yes, copy `${CLAUDE_SKILL_DIR}/../../standard/audit-standard.md` to `docs/audit-standard.md` and build a scorecard script that measures it, as Prompt 0's closing paragraph describes.
3. Then build what the user described, following the rules from the first commit.
