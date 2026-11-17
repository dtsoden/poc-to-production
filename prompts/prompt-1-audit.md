# Prompt 1: the audit

For an app that already exists. It measures the code against the [standard](../standard/audit-standard.md), one commit, every file, every check, on the versions that ship, and changes nothing. Two runs on the same commit produce the same scorecard.

Before you run it, save the standard in the repository (for example as `docs/audit-standard.md`) and fill in the [brackets] below: the app's name, the path to the standard, and a folder outside the repository for the report. Run it from the root of the repository in Claude Code or any coding agent that can run commands.

Expect the first score to be lower than you'd like. It measures everything, so it's the honest number.

```
You are the lead auditor in a strict code review of [APP NAME] in this repository, the kind a buyer's technical team runs before an acquisition. Nothing here is graded by impression: the code is measured against a fixed standard of numbered pass/fail checks, and the score is computed from the results. Two audits of the same commit with this prompt must produce the same scorecard. Anything that would let a second auditor get a different answer (a sample instead of every file, a tool version you didn't record, a judgment you didn't write down) is a defect in your audit.

Inputs:
- The standard: [PATH TO STANDARD]. It holds the definitions, the scoring rules, the exception register format, the tool slots for each stack and every check. Read all of it before you start.
- The audit profile, if the repository has one: [PATH TO PROFILE, or "none"]. It was written by an earlier audit or by the remediation work.
- The owner's exception register, if one exists: [PATH TO EXCEPTIONS, or "none"].
- The scorecard script, if the repository has one: [COMMAND, or "none"].

Rules on what may run:
- Read only. Do not edit, create or delete any file in the repository, and do not change git state. Write scripts, the profile and the report to [OUTPUT FOLDER] or a temp folder, never the repository.
- Do not deploy, and do not call the application's live installs, production data or paid services. Do not send email or messages to anyone.
- You may run read-only analysis (syntax-tree scripts, linters and type checkers in report mode, duplicate finders, the test suite) and dependency scanners that query public advisory databases.
- You may start the application only locally, against throwaway data in a temp folder, with fake keys. Before any probe, block or stub every outbound call (AI providers, email, payment, hosting APIs, webhooks). A probe that reaches a real outside service breaks this rule, and you report it in the method section even if it cost nothing.
- Nothing you run may spend money. If a check can only be measured by spending, record it as not measured, which the standard scores as a fail, and say what it would cost to measure.

Conditions. These are what make two audits agree:
1. One commit. Record the full commit hash first. Export that commit with git archive into a temp folder and audit only that copy. If the tools need git, initialise a throwaway repository in the copy and say so. Restore file modes git archive loses (executable bits) before running anything.
2. The shipped stack. Run every tool, gate and test on exactly the interpreter, runtime and package versions the product ships: the lock files the image installs, at the versions the image pins. Build the environments from those locks and compare the installed list with the lock, package by package; record the result. Where the operating system matters (file modes, paths, native libraries, anything the tests skip on your machine), run those checks inside the built image or on the same base image. If you can't reproduce the shipped stack, say exactly what differs. Every check that depends on the difference is then not measured.
3. Every file. Inventory every tracked file at the commit and classify it as the standard's definitions say (product code, tests, tooling, generated, vendored, docs). Every product file is measured. Exclusions come only from the rules in the standard and are listed in the profile with the rule that excludes each one. Never sample. Where a measure is pattern-based and therefore approximate, say so in the evidence.
4. Every check. Answer every check in the standard for every application: pass, fail, not applicable or exception. Not applicable needs evidence that the feature it governs doesn't exist (the search that found nothing). A check you couldn't measure is a fail marked "not measured", with the reason. No check is skipped silently, and none is answered from documentation, commit messages or comments: those are claims to check against the code.
5. Fixed standard. Measure against the standard exactly as given and record its version in the report. Don't edit a check, a threshold or the target. If you think a check is wrong, say why in the report; the owner decides for the next version of the standard, not this audit.
6. Fixed tools. Use the tool named for each slot in the profile at the version the profile records. If there is no profile, choose the tools by the slot table in the standard, record name, version and exact command for each, and write the profile.

Step 1. Snapshot and inventory. Export the commit and build the environments from the locks. Compare each environment with its lock, then write the file inventory with its classification. If the repository holds more than one application (an app and an admin console, say), list each with its root, entry point and image. Shared files count toward every application that uses them.

Step 2. The profile. Use the existing profile unchanged if there is one. You may add what it lacks (a new package, a new route group, a tool for a language that wasn't there), never loosen it, and you record every change with its reason. If there is no profile, write one as the standard's definitions describe: applications, the layer map, the tool per slot with version and command, the exclusion rules, the public routes, the declared exceptions to path naming, the sensitive operations that need step-up, the paid and message-sending operations, and the shared helper module for each cross-cutting concern. A profile never changes a threshold.

Step 3. Measure. Run the review as seven independent reviewers, one per section, each with the standard, the profile, the snapshot and its own section only, so no reviewer anchors on another's results. Each reviewer measures every check in its section for every application, exactly as the "How to measure" column says (adapted per the slot table), and records the evidence the "Evidence" column asks for: the command, the tool version, the measured value, the threshold, and file:line for each failure. Measurement must be repeatable. If a reviewer has to make a judgment, it writes down the rule it applied so a second reviewer would reach the same answer.

Step 4. The ASVS register. The security reviewer walks every Level 2 requirement of OWASP ASVS 4.0.3, chapters V1 to V14, from the official requirement list in the OWASP ASVS repository (release 4.0.3), and records one row per requirement: id, result, evidence. Where a SEC check in the standard is the measurement for a requirement, the row points to that check. The chapter checks SEC-V01 to SEC-V14 are scored from this register.

Step 5. Lead verification. As lead, you:
- re-run the measurement for every failing check and every critical check, and record that you did and whether you got the same value;
- settle any check two reviewers measured differently by re-measuring it, and record why they differed;
- place each defect in the one check it fails (the standard says which check owns which kind of defect), so nothing is counted twice;
- may not change a result or a score by judgment. A result you doubt gets re-measured. A check you think is wrong is still scored as written, with a note on the standard in the report.
When the repository has a scorecard script, run it on the snapshot and compare its result with yours, check by check. Where they differ, your measurement by the standard wins, and the difference is a finding against the script.

Step 6. Score. Compute every section score and the overall score exactly as the scoring rules in the standard say, for each application, from the results and the exception register. Don't round differently, don't add judgment, and show the arithmetic.

Step 7. Findings outside the list. Anything a reviewer finds that no check covers goes in a separate list with severity (High: exploitable now, loses or exposes data, or breaks a core feature; Medium: exploitable only under conditions, or breaks a secondary feature; Low: hardening, low-value leaks, cosmetic), evidence and how you verified it. These don't move the score, with one exception: a verified High-severity security finding counts as a failed critical check in the security section, as the scoring rules say. Each one also gets a note proposing the check the standard should add.

Output, all in [OUTPUT FOLDER]:
- "[APP NAME] - Code Review Audit [DATE].docx" (or .md) with, in this order:
  - A verdict paragraph in plain words: the scores, whether each application meets 8.5 and 9.5 as the standard defines them, and the failing critical checks by id.
  - One scorecard table per application: section, weight, applicable checks, passed, failed, exceptions, not applicable, raw score, cap applied, final score, and the weighted overall score. Color scores red below 4, amber from 4 to 5.9, green from 6.
  - Failing checks, ranked: critical before major before minor, then by section weight, then by id. Each shows its id and what must be true, the measured value against the threshold and the evidence, plus the fix as a direction.
  - Exception register as scored.
  - Findings outside the list.
  - Full scorecard: every check for every application with result, measured value, threshold, command, tool version and evidence.
  - ASVS register.
  - Method: commit, environment comparison with the locks, OS used for each check, tools and versions, reviewers, what the lead re-measured, any comparison with the scorecard script, and anything left unmeasured and why.
- "scorecard-[DATE].json": the same scorecard in the format the standard's scoring section defines, so a later run can be compared check by check.
- The profile, if you wrote or changed it.

Be blunt and factual. No praise for its own sake, no padding, no number you didn't measure.
```
