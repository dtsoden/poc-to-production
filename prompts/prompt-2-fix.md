# Prompt 2: the plan and the fix

Run it after Prompt 1. Its scope is every check the audit scored as failed or not measured, and nothing else unless you add it. Part A writes the plan and stops for your approval. Part B carries the plan out one phase at a time, starting with a scorecard script that measures every check, so done means the script is green.

Fill in the [brackets] with the paths Prompt 1 produced and the target you want (8.5 or 9.5, as the [standard](../standard/audit-standard.md) defines them).

```
You are the lead architect taking [APP NAME] from proof of concept to production. The scope is fixed: every check in the standard that the audit scored as failed or not measured, for every application. Nothing else is in scope unless the owner adds it. The goal is the same behavior on code where every one of those checks passes, measured by the same commands the audit used, reached in small steps that never break what users rely on.

Inputs:
- Standard: [PATH TO STANDARD].
- Audit report and scorecard: [PATH TO AUDIT] and [PATH TO SCORECARD JSON].
- Audit profile: [PATH TO PROFILE].
- Owner's exception register, if any: [PATH TO EXCEPTIONS, or "none"].

The standard, the profile's thresholds and the target are frozen for the whole plan. Don't raise the target, and don't edit a check to make it pass. Lessons from the run go in a "next version" notes file for the owner; they never change what this plan is measured against.

Read the standard and the audit in full, then read the code the failing checks cite until you understand how the application works today. Re-measure each failing check on the current code before relying on it, because the code may have moved since the audit.

Part A: the plan. Change no code in this part.

Write "[APP NAME] - Remediation Plan [DATE].docx" (or .md) in [OUTPUT FOLDER] covering:

1. The scope table. Every failing and not-measured check by id, per application, with its severity, the measured value and the threshold, and the phase that closes it. Every one appears exactly once. Checks under an owner exception are listed separately, with a question to the owner on whether to close them. Findings outside the list go in an appendix for the owner to bring into scope or leave out; they don't block done, except High-severity security findings, which are in scope.

2. Phase 0: measurement first. Before any fix:
   - Pin every gate and the test suites to the shipped stack: the runner builds or checks its environment from the locks the image installs and refuses to run on any other interpreter or package set.
   - Build the scorecard script: one command (for example `python tools/scorecard.py`) that measures every automatable check in the standard on the shipped stack, writes a scorecard in the standard's JSON format, computes the scores by the standard's rules, compares the result with the last recorded scorecard, and exits non-zero if any check fails or any check that passed before now fails. It reads the thresholds from the standard and the tools and lists from the profile, so neither is copied into code. Checks that need a person (a browser walk, a reading of a document) get a written procedure, a place to record the evidence, and a date; the script reports them as failing until the evidence for the current commit is recorded. Aim for at least 85 percent of the checks automated, and list the manual ones.
   - Run the script on the audited commit and reconcile it with the audit, check by check. Every difference is either a bug in the script (fix it) or an error in the audit (record it with evidence). The script is ready when it agrees with the audit on every check or every difference is explained.
   - Commit the profile, the standard and the script. Make the release build call the script.
   Phase 0 closes the delivery checks it touches (shipped-stack parity, the binding gate) and gives every later phase its proof.

3. Defects first. Each failing check that is a defect (a security hole, lost or exposed data, a broken feature) gets the smallest safe fix and the test that proves it, in the earliest phases, critical checks before major ones.

4. Target design, written only as far as the failing checks need it, each part naming the check ids it serves:
   - Layers and the one direction of dependency between them, the package layout that enforces them, and the boundary contracts the import checker will hold.
   - What moves where: each oversized file and function split into named modules, with the list of which functions go where (long lists in an appendix).
   - Data layer: one connection helper, a repository per area, one schema with numbered migrations.
   - Configuration in one settings module.
   - Master classes and shared modules: every duplicate and parallel group the checks found, mapped to one base class or shared module with its name, its public methods, the subclasses, what each one overrides, and the copies it replaces with file:line.
   - API contract: methods, status codes, one error format, models validated at the edge, pagination, idempotency, versioning, the generated description.
   - Security and authentication model, against the ASVS register rows that failed.
   - Front end and design system: shared modules, types, tokens, one definition per component, the accessibility fixes.
   - Delivery: gates, image, boot check, rollback image, logs, error tracking.
   Where the standard's tool slots offer a choice, pick one and say why.

5. The phases. Ordered, grouped into tracks, each small enough to review in one sitting (small = up to half a day, medium = one to two days; split anything larger). For each phase:
   - the check ids it closes, per application;
   - the exact measurement that proves each one: the scorecard script command for those ids, or the manual procedure;
   - what changes, which tests guard it (write characterization tests first wherever behavior isn't already covered), how to verify it on a live test install, and how to roll it back.
   No phase changes what the application does for its users, except defect and security fixes, accessibility fixes and new sign-in protections; name each exception in the phase that makes it. Phases that touch the same files run one after another, never in parallel branches. A shared module is written once, in one phase, before any phase that depends on it starts. Say whether feature work pauses during the plan or continues on the new structure only, and ask if it isn't clear.

6. The repository standards file (CONTRIBUTING.md, AGENTS.md or CLAUDE.md) that any developer or AI agent must follow: it points to the standard as the definition of done for every change, says where new code goes and what each layer may import, when to extend a base class instead of writing a new function, "search for an existing helper or component before writing a new one", and "run the scorecard script before you call anything finished". Short, specific and checkable.

7. Definition of done for the plan:
   - On the release commit, the scorecard script, run on the shipped stack, reports every in-scope check as passing (or under a recorded owner exception), with the manual evidence recorded for that commit, and no check that passed at the audit now failing.
   - The computed scores meet the target the owner sets: [TARGET: 8.5 or 9.5, as the standard defines them].
   - Then one confirmation audit: a fresh agent runs Prompt 1 on that commit, once. If it finds a check failing that the script passed, that is a bug in the script or the code. Fix both and re-run the script, then confirm only the checks involved. It does not start a new round of planning. If the checks involved still fail after that, stop and give the owner a ship or no-ship decision with the remaining list. There is no second confirmation audit.

8. The shortest path. Also list the smallest set of failing checks whose closing reaches the target, with its phases and size, so the owner can choose that as the scope instead of every failing check.

9. Questions for the owner, at the end: anything only they can answer (infrastructure, backups, integrations, brand colors, paid services, exceptions they want to keep), each with what it blocks and the cost of each answer in checks and points.

There is no new audit until every phase is done. Stop after Part A and wait for approval.

Part B: carrying it out. Only after the plan is approved, and one phase at a time unless the owner says to run a whole track.
- Start each phase by restating its definition of done: the check ids it closes and the scorecard command that proves them. Nothing else joins the phase once it starts.
- Before each phase, make a rollback point (a git tag or branch). Keep the restore points the owner names until the plan is finished.
- Never add anything that spends money (hosted CI, paid services, cloud resources) without asking first. Gates run locally, or on free infrastructure the owner approves.
- Checks on a live test install must not send email or messages to the owner, to customers or to real companies unless the check is about that email; say so first if it must.
- Write the missing characterization tests first and watch them pass on the old code.
- Make the changes for that phase only, following the standards file. Delete each copy you replace. A short overlap is allowed only where a live rollout needs it (an old route alias, an old signature version, a re-export while callers move), and each one gets a named removal phase.
- Run the scorecard script on the shipped stack. The phase is done when its checks pass and nothing that passed before fails. A check that still fails after two attempts is not "too large for this run": stop and give the owner the options (a different approach, an owner exception, or a later phase), and wait. Run the full test suite and every gate; all must pass. Never record a new baseline to make a gate pass.
- Give every long-running job (a test suite, a build, a helper agent) a hard time limit and watch it while it runs. A helper that hasn't reported within its limit is stopped and its work redone, never waited on.
- After each release, use the main flows on a live test install the way a person does, and look at what the screens show as well as the status codes. Design changes and new rules break things tests don't see.
- Work only in the repository and the scratch folders you were given. Don't leave worktrees or copies elsewhere, and if files you deleted come back (a sync tool restoring them, for example), stop and find the cause before building.
- Commit the phase with a message that says what moved and why, with the ids of the checks it closed.
- Read the full output of the script, the suites and every gate before you report. Report what changed, the scorecard result for the phase's checks, the test results and anything unexpected in one message, then stop and wait before the next phase.
- At the end of each track, review the tests: merge tests that cover the same behavior through code that is now one shared module, delete tests that read source code as text or only protected a move that is finished, and report the test count before and after. The safety-net tests make the count go up first; consolidation brings it back down.
- When the last phase is done, run the scorecard script on the release commit, record the result next to the audit's, then run the confirmation audit as item 7 says.
```
