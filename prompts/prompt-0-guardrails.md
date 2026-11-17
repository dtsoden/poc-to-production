# Prompt 0: guardrails for your first prompt

Use this and your app is Born Governed™: its rules exist before line one.

For a new project. Describe what you're building in your own words, the way you normally would, then paste this after it in the same first prompt. The agent writes the rules into the repository's instructions file in the first session, so every later session follows them too.

It's the [standard](../standard/audit-standard.md) written as rules to build by rather than checks to fail. If you want the measurement as well, save the standard in the repository as `docs/audit-standard.md` and ask the agent in the first session to build a scorecard script that measures it, so every session ends with a number.

```
Build this to production standards from the first commit, even though it starts as a proof of concept. These rules apply to every task in this project, now and later. In the first session, write them into the repository's agent instructions file (CLAUDE.md, AGENTS.md or CONTRIBUTING.md) so every future session follows them.

Structure
- Layers: interface (routes, pages, CLI), services (business rules), domain (pure types and rules), data access (repositories and one database helper), gateways (clients for outside services). Dependencies point one way. Enforce it with an import-boundary checker (import-linter, dependency-cruiser) from the first commit. No import cycles.
- Limits: functions 60 lines or fewer and none over 100, route handlers 40 or fewer, files 800 or fewer, complexity 20 or less. Split before you reach them.
- One schema with numbered migrations from day one. SQL only in the data layer, values always bound as parameters, table and column names only from fixed lists in code.
- Configuration read in one settings module, from the environment. Settings an admin changes in the app sit behind the same module.
- No mutable global state. Every cache has a size limit. Slow work never runs on the request path: start a job and report its progress.

One way to do each thing
- Before writing a helper, search for an existing one. Keep one shared module each for AI calls and parsing their output, HTTP calls (timeouts, retries only for calls that are safe to repeat, and a server-side request forgery guard on any URL a user can influence), database access, time (store UTC), settings, errors, retries, text and URL helpers, escaping and email.
- When two things are alike, write one base class with overrides, never a copy.
- Status values and repeated strings are named constants or enums. The front end gets them from a file generated from the server's enums.
- If there are two applications (say an app and an admin console), shared code lives in one shared package both import.

API
- REST with correct methods and status codes (RFC 9110). One error format everywhere (RFC 9457 problem details). Request and response models validated at the edge, with limits on every field. Paging with a maximum on every list. An Idempotency-Key on every POST that costs money or sends a message. An OpenAPI description generated from the code and checked in. Versioned paths for anything another system calls.

Security
- Deny by default: every route requires sign-in and a role unless it is on a short written list of public routes. Admin actions check the role on the server.
- OWASP ASVS 4.0.3 Level 2 is the checklist. Passwords per NIST SP 800-63B: at least 12 characters, a breached-password check, a slow hash. Session cookies Secure, HttpOnly, SameSite and __Host- prefixed, with idle sign-out and CSRF protection. Passkeys available as a second step. Sign-in throttling that slows attackers and never locks out the right person.
- Output escaped by default. A Content-Security-Policy without unsafe-inline from the first page, plus the standard security headers.
- Secrets never in code, git history or logs, encrypted at rest, with the key from the environment.
- Uploads checked by content and size (zip bombs included), request size limits, per-user rate limits on expensive routes. No error text from an outside service shown to users. Every server error logged with a reference the user can quote.

Front end
- ES modules only: no globals, no inline scripts, no inline event handlers. Types through TypeScript strict or JSDoc checked by tsc. One API client module every page uses. ESLint and Prettier.
- Design tokens for color, type, spacing, radius and shadow. One definition per component. No inline styles and no CSS inside scripts. stylelint. A living style guide page.
- WCAG 2.2 AA for contrast, labels, keyboard use and focus, with axe run on every page in the tests.

Tests and delivery
- Pin everything: lock files with hashes, and the tests and gates run on the same interpreter and package versions the shipped image uses, inside that image or its base image, from a clean checkout of the commit, never only on the developer's machine.
- Gates from day one, at zero findings: linter, formatter, strict type checker, import boundaries, duplicate finder, secret scan over the full history, dependency audit, image scan and Dockerfile lint. A baseline may only go down.
- Tests ship with every feature: mostly unit tests, then integration tests, then a few browser tests. Branch coverage at 85 percent or more. Tests never call real outside services or send real email.
- Every test passes alone and in any order, and never depends on timing, background work or another test's leftovers. A test that fails now and then is a bug report: find the cause before anything else.
- Images run as a non-root user, with base images pinned by digest. A health endpoint that reveals nothing private. Structured JSON logs. Error tracking with personal data stripped. A scheduled rescan of the shipped images for newly published vulnerabilities.
- Gates run locally, through a pre-push hook and the release build. Don't add hosted CI, paid services or anything else that spends money without asking me first.

How we work
- A feature is done when it works, its tests pass and every gate passes. Never lower a threshold or record a new baseline to get to green. If a rule seems wrong for a case, stop and ask me.
- A rule that would change what users see or what another system sends becomes a written exception I approve, with a compensating control and a review date. Never change behavior to pass a check without asking.
- Give every long job (a test suite, a build, a helper agent) a time limit and a visible watch, and tell me what is running. Agents working in parallel work on separate files and merge the main branch before they finish. Full test suites run one at a time.

How a task starts and ends
- Before you start any task, write its definition of done in a few lines: what will be true when it's finished, and which tests, gates or checks prove it. Show it to me before you build. The task ends when that is met, and the definition doesn't grow during the task.
- Build only what the current task asks for. New ideas, refactors, polish and anything a review finds that these rules don't cover go into BACKLOG.md for me to schedule. They never extend the task you're on.
- These rules don't change during a task. If one turns out to be wrong or missing, write the proposed change in BACKLOG.md under "Proposed rules". It applies from the next task, and only after I approve it.
- Work on a branch. Iterate with the fast local checks, then run the full suites, the gates and the build once when the task is done, not after every edit.
- If the same test, gate or check still fails after two fix attempts, stop. Tell me what you tried and the options: a different approach, a written exception, or the backlog. Don't keep trying.
- Never regenerate a whole document or AI-written output to fix one problem in it. Fix it in place. Names, numbers, dates and prices come from code or data, not from the model's prose. A review of AI-written text doesn't have to reach zero findings: fix what is wrong, and put style polish in the backlog.
- Call a task done only after reading the full output of every test and gate run. Report everything in one message: what is done, what failed, what went to the backlog. Then stop. Nothing merges to the main branch until I've reviewed it and said yes.
```
