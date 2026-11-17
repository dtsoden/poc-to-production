# From proof of concept to production

[![Born Governed](https://img.shields.io/badge/Born-Governed-2B88D8)](https://github.com/dtsoden/poc-to-production)

**Born Governed™: apps whose rules exist before line one.**

Three prompts and one written standard for building apps with an AI coding agent. Start a new app with Prompt 0 and it's Born Governed: the standard goes into the repository before the agent writes any code, and every session after that builds to it. That's governance as code from the first prompt, and it's the cheapest way to get an app to production.

If you already have an app that wasn't Born Governed, the same standard audits it and plans the fixes.

If this saves you a rewrite, star the repository. It's how I know it helped, and it's how the next person finds it before they need it.

## Install it as a Claude Code plugin

In Claude Code, run:

```
/plugin install poc-to-production --marketplace dtsoden/poc-to-production
```

On Claude Code older than 2.1.275, add the marketplace first, then install:

```
/plugin marketplace add dtsoden/poc-to-production
/plugin install poc-to-production@dtsoden
```

Then use it from any repository:

| Command | What it does |
|---|---|
| `/poc-to-production:guardrails a CRM for my sales team` | For a new project: writes the build rules into the repository's instructions file, then builds what you described to them. |
| `/poc-to-production:audit` | For an existing app: scores it against the 191 checks and changes nothing. |
| `/poc-to-production:fix 8.5` | After the audit: plans the fixes, waits for your approval, then makes them one phase at a time. |

If you use Cursor, Copilot or another agent, copy the prompts from the files below instead. They work in any coding agent that can run commands.

## What's here

| File | Use it when |
|---|---|
| [Prompt 0: guardrails](prompts/prompt-0-guardrails.md) | You're starting a new project. Paste it after your first prompt. |
| [Prompt 1: the audit](prompts/prompt-1-audit.md) | You have an app already. It measures the code and changes nothing. |
| [Prompt 2: the plan and the fix](prompts/prompt-2-fix.md) | After Prompt 1. It plans and makes the fixes, one phase at a time, with your approval. |
| [The standard](standard/audit-standard.md) | Always. All three prompts are built on it. |

## Start Born Governed

Describe your project to the agent in your own words, then paste [Prompt 0](prompts/prompt-0-guardrails.md) after it in the same first prompt. The agent builds to the rules from the first commit, writes them into the repository's instructions file (CLAUDE.md, AGENTS.md or CONTRIBUTING.md) so every later session follows them, and runs the gates before it calls anything done.

It makes the first version a little slower. It also means the proof of concept that takes off is already most of the way to a product.

### Show it's Born Governed

If your app was built from Prompt 0, add the badge to its README:

```markdown
[![Born Governed](https://img.shields.io/badge/Born-Governed-2B88D8)](https://github.com/dtsoden/poc-to-production)
```

## Fixing an app you already have

A vibe-coded app can work and still be a long way from production. I took one of mine from a score of 3.6 out of 10 to 8.8 against this standard, and the story of what that cost is in my LinkedIn article, [Learn from my vibe-coding bill: production cost me twice](https://www.linkedin.com/pulse/learn-from-my-vibe-coding-bill-production-cost-me-twice-david-soden-wpvse/). The short version: almost two weeks and about twice the build cost, against an afternoon of guardrails up front.

1. Save [the standard](standard/audit-standard.md) in your repository, for example as `docs/audit-standard.md`.
2. Run [Prompt 1](prompts/prompt-1-audit.md) from the root of the repository. It audits one commit, every file, on the versions your app actually ships, and writes a report and a scorecard outside the repository.
3. Run [Prompt 2](prompts/prompt-2-fix.md). It writes a plan and stops. When you approve it, it builds a scorecard script first, then fixes the failing checks one phase at a time and stops after each one.
4. Done is the scorecard script reporting every in-scope check as passing. A fresh agent then runs Prompt 1 once to confirm.

Expect the first audit score to be lower than you'd like. It measures everything, so it's the honest number.

## How the standard scores

The standard has 191 numbered checks in seven sections. Each check has a threshold, the command that measures it and the evidence to record, so two audits of the same commit agree. A check nobody measured counts as failed.

| Section | Weight | Checks |
|---|---|---|
| Architecture, layering and efficiency | 20% | 26 |
| Reuse and one source of truth | 15% | 22 |
| API design | 10% | 20 |
| Security, authentication and error handling (with OWASP ASVS 4.0.3 Level 2) | 20% | 52 |
| Front end, scripts and page structure | 10% | 19 |
| CSS, design system and accessibility (WCAG 2.2 AA) | 10% | 23 |
| Tests, tooling and operations | 15% | 29 |

Checks are critical (3 points), major (2) or minor (1). A section's score is the share of its points that pass, out of 10, and failed critical checks cap it. There are two targets:

- 8.5, ready for a strict review: overall 8.5 or more, no section under 8.0, every critical check passing.
- 9.5, a strict reviewer passes it as it stands: overall 9.5 or more, every critical and major check passing.

When a check can only pass by changing how your app behaves for its users, you can record an owner exception with a reason, a compensating control and a review date. It earns half credit. The details are in [the standard](standard/audit-standard.md).

The checks are written for a Python or TypeScript back end and a JavaScript or TypeScript front end. Section A.4 of the standard maps each tool slot to other stacks, and the thresholds stay the same.

## Rules the prompts follow

- Nothing that spends money gets added without asking: no hosted CI, paid services or cloud resources. Gates run locally.
- The audit is read only and never calls live installs, real data or paid services.
- Fixes change nothing users rely on, apart from defect, security and accessibility fixes, and each of those is named in the plan.
- Every long job gets a time limit and a watch, and full test suites run one at a time.

## How a run ends

AI coding agents can loop: build, review, find something new, rebuild, for weeks. I lost about three weeks to loops like that on the app behind these prompts, so every prompt now has a finish line that can't move:

- The standard and the target are frozen for the run. Lessons from the run go to a notes file or the backlog and apply to the next run, after you approve them.
- Done is arithmetic: the scorecard script passes, or the task's tests and gates pass. No reviewer has to come back with zero findings, and AI-written text is fixed in place, never regenerated.
- Anything found outside the checks goes to a backlog. It doesn't reopen the run unless it's a high-severity security problem.
- Two failed attempts at the same check, test or gate and the agent stops and asks you: a different approach, a written exception, or later.
- One confirmation audit at the end, never a second. If it still disagrees, you get a ship or no-ship decision with the list.
- The agent reports everything in one message and stops. Nothing merges, and no next phase starts, until you say yes.

## License

[CC BY 4.0](LICENSE). Use it, change it and share it, with credit to David Soden. Born Governed™ is David Soden's name for apps built to a written standard from the first prompt.
