# From proof of concept to production

Three prompts and one written standard for building apps with an AI coding agent. Use them to build an app right from the first commit, or to audit and fix one that was vibe coded.

A vibe-coded app can work and still be a long way from production. I took one of mine from a score of 3.6 out of 10 to 8.8 against this standard, and the story of what that cost is in my LinkedIn article, [Learn from my vibe-coding bill: production cost me twice](https://www.linkedin.com/pulse/learn-from-my-vibe-coding-bill-production-cost-me-twice-david-soden-wpvse/).

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

## Starting something new

Describe your project to the agent in your own words, then paste [Prompt 0](prompts/prompt-0-guardrails.md) after it in the same first prompt. The agent builds to the rules from the first commit, writes them into the repository's instructions file (CLAUDE.md, AGENTS.md or CONTRIBUTING.md) so every later session follows them, and runs the gates before it calls anything done.

It makes the first version a little slower. It also means the proof of concept that takes off is already most of the way to a product.

## Fixing an app you already have

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

## License

[CC BY 4.0](LICENSE). Use it, change it and share it, with credit to David Soden.
