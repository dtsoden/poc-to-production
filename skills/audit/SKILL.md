---
name: audit
description: Audit an existing app against the poc-to-production standard of 191 numbered checks and score it, changing nothing. Use when someone wants to know how far a vibe-coded app or proof of concept is from production.
argument-hint: "[output folder]"
disable-model-invocation: true
---

Read `${CLAUDE_SKILL_DIR}/../../prompts/prompt-1-audit.md` and carry out the prompt inside its code block on the repository in the current working directory.

Fill its placeholders like this, and tell the user what you chose before you start:

- [APP NAME]: the application's name from the repository (its README, package manifest or folder name).
- [PATH TO STANDARD]: `docs/audit-standard.md` if the repository has one, otherwise `${CLAUDE_SKILL_DIR}/../../standard/audit-standard.md`. Do not copy the standard into the repository: the audit is read only.
- [PATH TO PROFILE], [PATH TO EXCEPTIONS], the scorecard script: use the ones the repository has, or "none".
- [OUTPUT FOLDER]: $ARGUMENTS if given, otherwise a new folder named `poc-to-production-audit-<date>` next to the repository (outside it). Ask before creating it if the parent folder isn't one the user works in.
- [DATE]: today's date.

The prompt's rules on what may run are binding: read only, no live systems, nothing that spends money or sends a message. If a check needs something you can't do under them, score it as the prompt says.

When you finish, give the user the scores per application and the path to the report, and tell them the next step is `/poc-to-production:fix`.
