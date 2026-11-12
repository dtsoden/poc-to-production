---
name: fix
description: Plan and carry out the fixes for every check a poc-to-production audit failed, one approved phase at a time, until a scorecard script shows them passing. Use after /poc-to-production:audit.
argument-hint: "[8.5 or 9.5] [audit folder]"
disable-model-invocation: true
---

Read `${CLAUDE_SKILL_DIR}/../../prompts/prompt-2-fix.md` and carry out the prompt inside its code block on the repository in the current working directory.

Arguments: $ARGUMENTS

Fill its placeholders like this, and tell the user what you chose before you start:

- [TARGET]: 8.5 or 9.5 if the arguments name one, otherwise 8.5.
- [PATH TO AUDIT], [PATH TO SCORECARD JSON], [PATH TO PROFILE]: from the audit folder named in the arguments, otherwise the newest `poc-to-production-audit-*` folder next to the repository. If there is no audit, stop and tell the user to run `/poc-to-production:audit` first.
- [PATH TO STANDARD]: `docs/audit-standard.md` if the repository has one. If it doesn't, copy `${CLAUDE_SKILL_DIR}/../../standard/audit-standard.md` there as part of Phase 0, because the scorecard script reads it.
- [PATH TO EXCEPTIONS]: the repository's exception register, or "none".
- [OUTPUT FOLDER]: the audit folder.

Part A ends with the plan and a stop. Do not start Part B until the user approves the plan, and then run one phase at a time, stopping after each, unless the user says to run a whole track.
