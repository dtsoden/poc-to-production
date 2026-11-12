# The standard: 191 checks for a production-ready app

Version 2.1. Save this file in your repository (for example as `docs/audit-standard.md`) and fill in anything in [brackets]. Prompt 1 measures your code against it, Prompt 2 fixes what fails, and Prompt 0 is the same standard written as rules to build by.

## A.1 Definitions

Application: a deployable unit with its own entry point and image. Each is scored on its own. A file used by two applications counts toward both.

Product code: every tracked file that ships in an application's image or is served to a browser, minus tests, tooling, docs, generated files and vendored third-party code. Generated means produced by a tool and marked as such (a header, or a path the profile lists with the command that generates it). Vendored means third-party code copied in, listed in the profile with its source and version. Nothing else may be excluded.

Layers: interface (route handlers, page handlers, CLI entry points), services (use cases and business rules), domain (pure types and rules with no input or output), data access (repositories and the database helper), gateways (clients for outside services). The profile maps each package or folder to one layer.

Handler: a function bound to a route. Function length: lines from the definition line to the last line of the body, docstring included. File length: physical lines.

Severity and weight: critical 3, major 2, minor 1.

One owner per defect: a defect counts against the check whose "what must be true" it breaks. Where two checks could hold it, the check in the higher-weight section owns it, and within a section the lower id. The other check is still measured on its own terms.

Results: PASS, FAIL, NA (the feature doesn't exist, with the search that shows it), EXC (an owner exception in the register). A check that wasn't measured is a FAIL marked "not measured".

The profile: a file recording, per application, its root and image, the layer map, the tool for each slot in A.4 with version and command, the exclusions with their rule, the public routes, any path segments allowed as action names, the sensitive operations, the paid and message-sending operations, and the shared module for each cross-cutting concern (AI calls, HTTP, database, time, settings, errors, retries, text and URL helpers, escaping). It never changes a threshold.

## A.2 Scoring

For each application and section:

- P = (sum of the weights of passing checks + half the weights of EXC checks) / (sum of the weights of all checks except NA).
- Raw score = 10 x P, rounded half up to one decimal.
- Caps: one failed critical check caps the section at 6.0; two or more cap it at 4.0. Any failed major check caps it at 9.4. A verified High-severity security finding outside the list counts as one failed critical check in the security section. EXC checks never trigger a cap.
- Final section score = the lower of the raw score and the cap.

Overall score = 0.20 x architecture + 0.15 x reuse + 0.10 x API + 0.20 x security + 0.10 x front end + 0.10 x CSS and accessibility + 0.15 x tests and operations, rounded half up to one decimal.

The targets:

- 8.5, ready for a strict review: overall at least 8.5, no section below 8.0, and every critical check passing in every section. A critical check under an owner exception counts as not passing here.
- 9.5, a strict reviewer passes it as it stands: overall at least 9.5, and every critical and major check passing in every section with no exceptions on either.

Scorecard format, one record per check per application: `id`, `app`, `section`, `severity`, `result`, `measured`, `threshold`, `command`, `tool_version`, `evidence` (file:line list or a path to the output), `verified_by` (reviewer, lead, script), `note`. Plus one record per section with the arithmetic, and one with the overall score and the targets met.

## A.3 Owner exceptions

An owner may decide that a check won't be met, for a business reason. Each exception is a row in the register: check id, application, who decided (name), date, why, the compensating control, how that control is measured, and a review date. The auditor measures the compensating control. If it holds, the check scores EXC (half credit, no cap). If it doesn't, or the register row is incomplete, the check scores FAIL. An exception expires on its review date.

## A.4 Tool slots by stack

The checks name a slot; the profile names the tool. Any other stack swaps in its own tool for each slot and keeps the threshold. If a stack has no tool for a slot, write a script that implements the measure exactly as the check describes. Keep it with the profile and record it as the tool.

| Slot | Python | TypeScript and JavaScript | Other stacks |
|---|---|---|---|
| Import boundaries | import-linter | dependency-cruiser or eslint-plugin-boundaries | ArchUnit (Java), ArchUnitNET (.NET), go-arch-lint |
| Import cycles | syntax-tree import graph with Tarjan's algorithm, including imports inside functions | madge --circular | the boundary tool's cycle rule |
| Function length and complexity | syntax-tree script; radon cc | ESLint max-lines-per-function and complexity | lizard (most languages) |
| Duplicates | normalized syntax-tree hashing plus difflib ratio; jscpd | jscpd | jscpd, PMD CPD |
| Linter | ruff | ESLint (eslint:recommended, typescript-eslint recommended) | the language's standard linter |
| Formatter | ruff format or black | Prettier | gofmt, rustfmt, dotnet format |
| Type checker | mypy or pyright, strict | tsc strict, or tsc with checkJs over JSDoc | the compiler |
| Dependency audit | pip-audit on the lock | npm audit or osv-scanner on the lock | osv-scanner |
| Static security analysis | semgrep (p/owasp-top-ten), ruff S rules | semgrep, eslint-plugin-security, eslint-plugin-no-unsanitized | semgrep |
| Secret scan | gitleaks over full history | gitleaks | gitleaks |
| Image scan and Dockerfile lint | trivy, hadolint | trivy, hadolint | trivy, hadolint |
| CSS lint | not applicable | stylelint with stylelint-config-standard | stylelint |
| Accessibility | axe-core through Playwright | axe-core through Playwright | axe-core |
| API description lint and contract test | spectral; schemathesis | spectral; schemathesis or dredd | spectral; schemathesis |
| Dead code | vulture (confidence 80) | knip or ts-prune | the linter's unused rules |
| Coverage | coverage.py with branch coverage | c8 or istanbul with branch coverage | the language's coverage tool |

Probes that need a running application use the local copy from Prompt 1's rules: throwaway data, fake keys, every outbound call stubbed, and a test client or a local browser.

## A.5 The checks

191 checks: architecture 26, reuse 22, API 20, security 52 (38 specific and 14 ASVS chapters), front end 19, CSS and accessibility 23, tests and operations 29. "0" means none at all, not "few".

### Section 1: architecture, layering and efficiency (20 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| ARC-01 | Critical | A layer map exists and a boundary tool enforces it | Contracts cover 100% of product packages | Read the boundary tool config; list product packages; compare | Config path, packages covered of total |
| ARC-02 | Critical | The boundary contracts hold | 0 broken contracts | Run the import boundary tool | Tool output, version |
| ARC-03 | Major | No import cycles among product modules | 0 cycles, counting imports inside functions | Build the import graph from every import statement, including nested ones; find strongly connected components | Cycle count, largest cycle's modules |
| ARC-04 | Major | First-party imports sit at module top | 0 first-party imports inside functions; third-party lazy imports each carry a reason comment | Syntax-tree scan of import statements below module level | Count, file:line list |
| ARC-05 | Critical | Handlers hold no data access | 0 database driver, query builder or SQL string uses in the interface layer | Syntax-tree scan for driver calls and SQL keywords in string literals, per layer | Count, file:line |
| ARC-06 | Major | Data access lives only in the data-access layer | 0 driver or SQL uses outside it | Same scan, all other layers | Count per module |
| ARC-07 | Major | Services, domain and data access don't know the web framework | 0 imports of the web framework in those layers | Boundary contract or import scan for the framework packages (fastapi, starlette, flask, express, next/server and so on) | Count, file:line |
| ARC-08 | Major | Handlers call services only | 0 imports of repositories, gateways or the database helper from the interface layer | Boundary contract or import scan | Count, file:line |
| ARC-09 | Major | Handlers are thin | 0 handlers over 40 lines | Function length of every function bound to a route | List over the limit |
| ARC-10 | Major | Functions are short | 0 functions over 100 lines; at most 1% of functions over 60 | Function length over every product function and method; nested functions also count toward the parent | Counts, the 15 longest with file:line |
| ARC-11 | Major | Functions are simple | 0 functions with cyclomatic complexity over 20 | radon cc (Python), ESLint complexity rule (JS/TS), lizard elsewhere | Count, worst 10 |
| ARC-12 | Minor | Files are a readable size | 0 product files over 800 lines | Line count per product file | List over the limit |
| ARC-13 | Minor | Route modules are focused | 0 modules with more than 25 handlers | Route inventory grouped by module | Max per module |
| ARC-14 | Major | One schema with numbered migrations | 0 DDL statements outside migration files; the database records its schema version | Scan for CREATE, ALTER and DROP; read the migration runner | Count, schema version mechanism |
| ARC-15 | Major | Environment read in one place | 0 reads of environment variables outside the settings module | Scan for os.environ, os.getenv, process.env and equivalents | Count, file:line |
| ARC-16 | Major | No hidden module state | 0 global statements; 0 module-level mutable containers outside an allow-list in the profile, each with a reason and a bound | Syntax-tree scan of module-level assignments of list, dict, set and mutable constructors | Count, file:line |
| ARC-17 | Major | Processes are disposable | Job state and progress live in the database or a queue; after a kill and restart, every running job resumes or shows as interrupted | Start each job type on a throwaway instance, kill the process, restart, read job status | Job types tested, status after restart |
| ARC-18 | Major | No blocking calls in async code | 0 async functions that reach a blocking call (sync database driver, sync HTTP, smtplib, time.sleep, subprocess, queue.get, sync file reads of uploads, CPU-heavy library calls) without offloading; 0 async handlers with no await | Linter async rules (ruff ASYNC) plus a call-graph script from every async function through first-party code to the blocking list; Node: any *Sync call reachable from a handler | Count, call chain per case |
| ARC-19 | Major | The event loop stays responsive under load | p99 loop lag at most 100 ms while each long-running operation runs | Run each long operation with stubs on a local instance while a probe task measures loop lag | Lag per operation |
| ARC-20 | Major | No queries in loops | Every collection endpoint runs the same number of queries for 1 row and for 50 rows | Count queries per request (driver trace or ORM event) on seeded throwaway data | Query counts per endpoint |
| ARC-21 | Minor | Caches are bounded | 0 in-process caches without a size limit or expiry | Scan for lru_cache(maxsize=None), dict caches written in functions, Map caches in JS | Count, file:line |
| ARC-22 | Major | Dependencies are passed in | 0 assignments to another module's attributes at run time (wiring by patching modules) | Syntax-tree scan for assignments whose target is an imported module's attribute | Count, file:line |
| ARC-23 | Minor | Modules use each other's public names only | 0 uses of another module's private names (a leading _ in Python; non-exported names reached by any route in JS) | Syntax-tree scan of imports and attribute access | Count per module |
| ARC-24 | Major | Slow work runs off the request path | 0 handlers whose request waits on an AI call, an outside service or work measured over 2 s; such work starts a job and returns a status resource | Handler timing in the test suite with stubs that add the recorded latency; read handlers that call gateways | List of handlers, timings |
| ARC-25 | Minor | No dead code | 0 unused functions, classes or modules | vulture at confidence 80, knip or ts-prune | Count, list |
| ARC-26 | Minor | Code lives in packages | 0 product modules at the repository root other than declared entry points | List root-level source files | List |

### Section 2: reuse and one source of truth (15 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| RU-01 | Critical | No copied functions | 0 pairs of identical function bodies of 6 lines or more after normalizing (identifiers renamed, literals blanked, docstrings and comments removed), all languages | Normalized syntax-tree hashing for Python; jscpd strict mode for the rest | Pair count, list |
| RU-02 | Major | No near copies within an application | 0 pairs at 0.90 similarity or above, 6 lines or more | difflib SequenceMatcher ratio over normalized token sequences; jscpd with the same minimum | Pair count, list with ratios |
| RU-03 | Major | Little duplicated mass | Duplicated lines at most 1% of product lines | jscpd over all product code, minimum 6 lines and 50 tokens | Percentage, top clones |
| RU-04 | Major | One name, one meaning | 0 same-named functions in two or more modules with different bodies, apart from methods that override a common base | Syntax-tree inventory of function names per module, bodies compared | List |
| RU-05 | Major | One AI gateway | Provider SDKs (openai, anthropic and so on) imported in exactly one module | Import scan | Importing modules |
| RU-06 | Major | AI replies parsed in one place | 0 JSON parsing or code-fence stripping of model replies outside the gateway | Scan for json.loads, JSON.parse and fence regexes in modules that receive gateway text | Count, file:line |
| RU-07 | Major | One HTTP client | HTTP libraries imported only in the shared HTTP module and the guarded fetcher for user-given addresses | Import scan for requests, httpx, urllib.request and aiohttp, plus fetch or axios in server code | Importing modules |
| RU-08 | Major | One database helper | Driver connect calls in exactly one module | Scan for connect calls | Count, file:line |
| RU-09 | Major | One time module, one stored format | 0 direct now(), utcnow(), Date.now() or new Date() calls for stored or compared times outside the time module; every stored timestamp in one format and zone | Scan; inspect schema and a seeded database for time columns | Count, formats found |
| RU-10 | Major | Settings read through one accessor | 0 direct reads of the settings store outside the settings module | Scan for the settings table or file name outside it | Count, file:line |
| RU-11 | Major | One error hierarchy | 0 raises of framework HTTP exceptions outside the interface layer's error translator; product code raises the shared error types | Scan for raise HTTPException and equivalents | Count per layer |
| RU-12 | Major | One retry helper | 0 hand-written retry loops (a loop around a call with a catch and a sleep) outside it | Syntax-tree pattern scan | Count, file:line |
| RU-13 | Major | Statuses and roles are named | 0 string literals compared with a value that exists as an enum or constant member | Collect enum values; scan comparisons (==, !=, in, match, switch) against those literals | Count per value |
| RU-14 | Minor | Few magic strings | At most 10 string literals of 4 characters or more (not messages or user copy, not dict keys, not the entries of a module-level constant table) repeated in 4 or more modules outside the constants module | Syntax-tree string inventory, excluding docstrings, log and user messages, dict keys (dict literal keys, subscript indexes, the key given to get, pop or setdefault) and the entries of a module-level list, tuple, set or dict assigned to an UPPER_CASE name | Count, list |
| RU-15 | Major | Applications share code instead of copying it | 0 pairs at 0.90 similarity or above across application boundaries | The RU-02 measure across applications | Pair list |
| RU-16 | Major | Parallel modules share a base | 0 groups of 3 or more classes or modules exposing the same 3 or more public names (Jaccard 0.8 or more) without a common base class or protocol | Syntax-tree inventory of public names, clustered; check inheritance | Groups found |
| RU-17 | Minor | One escape helper per language | Exactly 1 HTML escape function per language | Scan for escape function definitions | Count per language |
| RU-18 | Minor | One helper per cross-cutting concern | Each concern in the profile (text, URLs, ids and references, money) has one module, and 0 hand-written copies of its idioms remain | For each concern, scan for the idioms the profile lists | Count per concern |
| RU-19 | Major | Server values reach the client from one source | 0 hand-copied lists of server enum values, limits or roles in client code | For each server enum, scan client code for 2 or more of its values in one literal | List |
| RU-20 | Minor | One rendering path per output format | HTML pages, emails, documents and spreadsheets each built through one builder or template system | List generators per format; check what they inherit or call | Generators per format |
| RU-21 | Minor | Prompt and copy text defined once | 0 pairs of prompt strings of 200 characters or more at 0.90 similarity; NA if the product calls no AI | String inventory with difflib | Pair list |
| RU-22 | Major | One implementation per business operation | 0 operations (create, suspend, delete and so on) implemented separately in two handlers; pages and API call the same service function | Route inventory traced to service calls | Operations with two paths |

### Section 3: API design (10 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| API-01 | Critical | A generated OpenAPI 3 description is committed and current | Regenerated document equals the committed one, 0 differences after canonical JSON sorting, with a fixed hash seed | Generate from the code at the commit; diff | Diff size |
| API-02 | Major | The description lints clean | 0 errors and 0 warnings | spectral lint with the spectral:oas ruleset | Output |
| API-03 | Major | Every operation is typed | 0 operations with an empty or untyped success schema; 0 bodies without a requestBody schema | Script over the document | Count, operation ids |
| API-04 | Critical | Behavior matches the description | 100% of operations exercised; 0 responses that don't validate against the description | schemathesis (or dredd) against a local instance with stubs and a seeded user per role | Operations run, mismatches |
| API-05 | Major | One error format | 100% of 4xx and 5xx bodies are RFC 9457 problem+json with type, title and status, including 404, 405, 413, 422, 429 and forced 500 | Probe each operation signed out, with a bad body, an unknown id, a wrong method and an oversized body; force a 500 in a test | Bodies that fail |
| API-06 | Major | Errors are documented | 0 observed status codes missing from the operation's documented responses | Compare API-04 and API-05 results with the document | Count, list |
| API-07 | Major | Safe methods are safe | 0 GET or HEAD handlers that write data, send a message, start a job or call a paid service | Run every GET with a database change counter and recording stubs | List |
| API-08 | Major | Methods match their meaning | 0 CRUD verbs (create, get, list, update, delete, remove, add, set, save) in paths; DELETE deletes; PATCH for partial updates; other action segments only if listed in the profile | Route inventory | Count, list |
| API-09 | Major | Status codes match their meaning | Creates answer 201 with Location; async starts 202 with Location of a status resource; deletes 204, or 202 when async; 0 exceptions | From the API-04 run | Deviations |
| API-10 | Major | Input validated at the edge | 100% of bodies, query and path parameters bound to typed models; unknown fields refused; every string has maxLength and every array maxItems | Script over the document plus one probe per model with an extra field | Fields without limits |
| API-11 | Major | Responses are filtered | 0 response models that pass through undeclared fields; a model that sends the handler's value unchanged by design counts as filtered when the test suite checks every answer it produces against the model and fails on a field the model doesn't declare | Read model config; a test that adds an extra field to a stored object and reads it back; for a deliberate pass-through, a test that the suite's guard fails on an undeclared field, and a passing suite run | Models allowing extras without that guard |
| API-12 | Major | Collections are paged | 100% of collection GETs take limit and cursor, enforce a documented maximum, use one envelope and page in the query, not in memory | Document review plus a probe with more rows than the maximum and a query trace | Non-conforming operations |
| API-13 | Major | Retried writes are safe | 100% of POSTs that create a billable resource, send a message or start paid work require an Idempotency-Key or reject a duplicate in flight; a repeat with the same key returns the first result with 1 effect | Probe each listed operation twice with recording stubs | Effects per operation |
| API-14 | Minor | Machine routes are versioned | 0 operations used by integrations outside a version prefix or header | Route inventory against the profile's integration list | List |
| API-15 | Minor | Old routes are announced | Every alias carries a Deprecation header and a Sunset header (RFC 8594) and has a removal date | Probe each alias | List |
| API-16 | Minor | One naming style | 0 paths or JSON field names outside the chosen case style | Script over the document | Count |
| API-17 | Major | Auth and limit responses are correct | 401 carries WWW-Authenticate; 403 only for a signed-in caller without permission; 429 carries Retry-After; 0 deviations | Probes | Deviations |
| API-18 | Minor | No duplicate endpoints | 0 pairs of operations that call the same service function with the same meaning, except announced aliases | Route inventory traced to service calls | Pairs |
| API-19 | Minor | Security is declared | securitySchemes present; every operation has a security requirement or an explicit public entry | Script over the document | Operations missing it |
| API-20 | Minor | Concurrent edits are detected | Shared editable resources support ETag and If-Match; a stale If-Match gets 412; NA if nothing is edited by more than one user | Probe | Resources checked |

### Section 4: security, authentication and error handling (20 percent)

Specific checks. Each one is also the evidence for the ASVS requirements it names in the register.

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| SEC-01 | Critical | Access is denied by default | Access control declared in one place so a new route is protected unless listed public; 0 non-public routes answer a signed-out request with anything but 401 or a sign-in redirect | Enumerate routes from the running app; probe each signed out | Routes probed, failures |
| SEC-02 | Critical | Admin functions check the role | 0 admin operations that succeed for an ordinary user (OWASP API5) | Probe every admin operation as each non-admin role | Failures |
| SEC-03 | Critical | Objects check the owner | 0 object operations that succeed for another user's object id (OWASP API1) | Probe every operation that takes an id as a second user | Failures |
| SEC-04 | Critical | No cross-site scripting | 0 script executions when a payload is stored in every free-text field and returned from stubbed AI replies, rendered on every page | Seed payloads, render every page in a headless browser, listen for execution; also semgrep XSS rules at 0 | Fields, pages, executions |
| SEC-05 | Critical | No injection | 0 queries whose text holds a value from input: values are bound as parameters, and a table or column name in the text comes only from a fixed allow-list in code; 0 shell=True, os.system, eval or exec with input | semgrep taint rules from input to the query text (p/sql-injection, p/command-injection, or local rules that do the same), ruff S602, S307 | Count, file:line |
| SEC-06 | Critical | No known vulnerable dependencies | 0 known advisories in the shipped locks | pip-audit on the lock; npm audit --omit=dev or osv-scanner on the lock | Output |
| SEC-07 | Major | The image is clean and hardened | 0 HIGH or CRITICAL fixable CVEs; non-root user; base image pinned by digest | trivy image --ignore-unfixed --severity HIGH,CRITICAL on the built image; read the Dockerfile | Counts, user, digest |
| SEC-08 | Critical | No secrets in code or history | 0 findings; every allow-list entry has a reason | gitleaks over the full history | Commits scanned, findings |
| SEC-09 | Major | Stored secrets are encrypted with a key kept apart | Secrets in the database are encrypted; the key is not on the same volume or in the same backup; the key is 32 random bytes or comes from a KDF | Read the key loading and backup code | Key location, derivation |
| SEC-10 | Critical | Passwords are stored safely | Argon2id, scrypt, bcrypt, or PBKDF2-HMAC-SHA256 at 600,000 iterations or more (OWASP Password Storage Cheat Sheet); constant-time compare; a dummy hash for unknown users | Read the code; a unit test reads the parameters | Algorithm, parameters |
| SEC-11 | Major | Password policy follows ASVS and NIST | Minimum 12 characters (ASVS 2.1.1); 64 or more allowed; no composition rules; breached-password check (ASVS 2.1.7) | Probe set-password with 11 characters, 64 characters and a known breached password | Results |
| SEC-12 | Critical | A second factor exists and admins need it | TOTP or WebAuthn available to every password account; required by default for admin and owner accounts on a fresh install | Fresh install with default settings, create an admin, sign in | Prompted or not |
| SEC-13 | Major | Sensitive operations ask again | 100% of the profile's sensitive operations (revealing secrets, adding or removing a sign-in factor, changing email, restoring backups, changing SSO, deleting accounts or installs) need a password or second factor within the last 15 minutes, and notify the account | Probe each with a session older than 15 minutes | Failures |
| SEC-14 | Critical | Session cookies are locked down | Every session cookie Secure, HttpOnly and SameSite Lax or Strict, whatever the request scheme or forwarded headers (ASVS 3.4.1 to 3.4.3) | Sign in over plain http and over https in a test client, with and without forwarded headers | Cookie attributes |
| SEC-15 | Major | Sessions end | Absolute lifetime at most 12 hours, idle timeout at most 30 minutes for admins; sessions revoked on sign-out and password change, server side | Probes with a moved clock | Results |
| SEC-16 | Major | Session tokens are strong and stored hashed | 128 bits or more of randomness; stored hashed; a new token at sign-in | Read the code; compare tokens before and after sign-in | Results |
| SEC-17 | Critical | Cross-site writes are refused | 100% of state-changing requests with a foreign Origin and a valid cookie refused (ASVS 4.2.2) | Probe every write operation | Failures |
| SEC-18 | Major | Guessing is throttled without locking out the owner | Sign-in, reset, second-factor and password re-checks throttled per account and per real client address; client address taken only from the trusted proxy; the right password still works from another address after 20 bad tries | Probe with rotating forwarded headers, then the right password from a new address | Results |
| SEC-19 | Major | Accounts can't be enumerated | Same message and median response time within 10% for known and unknown accounts on sign-in and reset, over 20 tries | Timed probes | Medians |
| SEC-20 | Major | Recovery is safe | Reset and invite links built from the configured address, never request headers; single use; expire within 1 hour; stored hashed | Probe with a foreign Host header; reuse a token; read storage | Results |
| SEC-21 | Major | SSO is checked end to end | state, nonce, PKCE S256, issuer and audience checked, algorithm allow-list without none or HS*, accounts linked only on email_verified, sign-up limited to allowed domains or groups; NA without SSO | Read the flow; probe with forged claims in a test | Missing items |
| SEC-22 | Major | Browser security headers are set | Every HTML response has an enforced CSP with default-src, script-src without unsafe-inline or unsafe-eval, object-src 'none', base-uri and frame-ancestors; HSTS max-age of 31536000 or more; nosniff; Referrer-Policy | Fetch every page route from a local instance; parse headers | Pages failing, missing directives |
| SEC-23 | Minor | Styles don't need unsafe-inline | style-src without unsafe-inline; 0 CSP violation reports during the browser test run | Read the policy; collect reports in the run | Violations |
| SEC-24 | Critical | Server-side fetches can't reach inside | 100% of server fetches of user-, admin- or third-party-supplied URLs refuse loopback, private, link-local and metadata addresses (IPv4 and IPv6), pin the checked address, re-check every redirect hop and allow only http and https | Probe with 127.0.0.1, [::1], 169.254.169.254, a name resolving to 10.0.0.1, a local redirect to 127.0.0.1 and ftp:// | Cases refused of total |
| SEC-25 | Major | Request size is capped | A body cap before authentication on every route, counting chunked bodies; per-route upload caps; archives checked for total uncompressed size and ratio before extraction | Send twice the cap, chunked, signed out; upload a zip bomb | Status, memory peak |
| SEC-26 | Major | Uploads are handled safely | Type checked by content; stored outside the web root under generated names; size capped (ASVS 12.1, 12.2) | Read the upload code; probe with a renamed executable | Results |
| SEC-27 | Major | Expensive operations are limited | 100% of operations that call a paid service or run longer than 1 s have a per-user rate limit | Profile list against the limiter config; probe each | Operations without one |
| SEC-28 | Major | Errors reveal nothing | 0 responses with stack traces, SQL, file paths or upstream error text, apart from a deliberate pass-through: an administrator's own connection test that hands back the upstream server's reply about the settings they entered, with a reference; every 5xx carries a reference that appears in the log | Force failures in tests; scan bodies; match references in captured logs | Leaks |
| SEC-29 | Major | No silent broad catches | 0 broad catches that neither log with the trace, re-raise nor return an error | ruff BLE001 and S110, an empty-catch scan for JS, read each remaining broad catch | Count, file:line |
| SEC-30 | Major | No unsafe retries | 0 automatic retries of a non-idempotent outside call (POST or DELETE, sending mail) without an idempotency key or a state check | List every retry site and its target | Sites |
| SEC-31 | Major | Outbound TLS is verified | 0 verify=False, unverified SSL contexts or SMTP without a verifying context | ruff S501 and S323; scan for ssl contexts | Count |
| SEC-32 | Major | Keys carry least privilege | Every API key and integration token has a scope; 0 operations a key can reach outside its scope | Probe each key type against every operation | Out-of-scope successes |
| SEC-33 | Major | Public endpoints say little | Public responses contain only documented fields: no install or customer names, disk, user counts or internal versions; API docs not public unless intended | Probe every public route | Fields exposed |
| SEC-34 | Major | Security events are audited | Sign-in, failed sign-in, role change, secret reveal, export, restore and delete each write an append-only audit record with actor, time and address | Perform each on a local instance; query the audit store | Events missing |
| SEC-35 | Minor | Logs hold no secrets | 0 occurrences of test passwords, tokens or keys in logs captured during the full test run | Seed known values; scan captured logs | Hits |
| SEC-36 | Minor | CORS is narrow | 0 wildcard or development origins with credentials in production settings | Read config; probe with a foreign Origin | Origins allowed |
| SEC-37 | Minor | Static security analysis is clean | 0 unsuppressed high-severity findings; every suppression has a reason | semgrep p/owasp-top-ten, plus ruff S or eslint-plugin-security | Output |
| SEC-38 | Minor | A threat model exists and is current | A written threat model or data-flow diagram covering every outside service found in RU-05 and RU-07 and every trust boundary | Compare the document with the gateway inventory | Services missing |

ASVS chapter checks. The register holds one row per Level 2 requirement of OWASP ASVS 4.0.3. Each chapter check passes when every applicable requirement in that chapter is answered pass or carries an owner exception. A requirement measured by a specific SEC check takes that check's result.

| ID | Sev | Chapter | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| SEC-V01 | Major | V1 Architecture, design and threat modeling | 0 failed applicable L2 requirements | Walk the chapter's requirements one by one from the 4.0.3 list; record pass, fail, NA or exception with evidence | Register rows |
| SEC-V02 | Critical | V2 Authentication | Same | Same | Same |
| SEC-V03 | Critical | V3 Session management | Same | Same | Same |
| SEC-V04 | Critical | V4 Access control | Same | Same | Same |
| SEC-V05 | Critical | V5 Validation, sanitization and encoding | Same | Same | Same |
| SEC-V06 | Major | V6 Stored cryptography | Same | Same | Same |
| SEC-V07 | Major | V7 Error handling and logging | Same | Same | Same |
| SEC-V08 | Major | V8 Data protection | Same | Same | Same |
| SEC-V09 | Major | V9 Communication | Same | Same | Same |
| SEC-V10 | Major | V10 Malicious code | Same | Same | Same |
| SEC-V11 | Major | V11 Business logic | Same | Same | Same |
| SEC-V12 | Major | V12 Files and resources | Same | Same | Same |
| SEC-V13 | Major | V13 API and web service | Same | Same | Same |
| SEC-V14 | Major | V14 Configuration | Same | Same | Same |

### Section 5: front end, scripts and page structure (10 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| FE-01 | Critical | No inline script | 0 inline script blocks with code and 0 inline event handler attributes, in pages, templates and HTML built in code | HTML parser over pages and templates; scan code strings for script tags and on* attributes | Count, file:line |
| FE-02 | Major | Scripts are modules | 100% of first-party scripts are ES modules | Scan script tags and files for type=module and import or export | Classic scripts |
| FE-03 | Major | No globals | 0 assignments to window or globalThis; 0 implicit globals | Scan; ESLint no-implicit-globals | Count |
| FE-04 | Major | Types are checked strictly | TypeScript strict, or tsc with checkJs and strict over JSDoc, covering 100% of first-party scripts; 0 errors | tsc --noEmit with the project config; compare its file list with all scripts | Errors, files outside |
| FE-05 | Major | any is rare | At most 1 explicit any (type, JSDoc tag or cast) per 1,000 lines | ESLint no-explicit-any count, plus a JSDoc tag scan | Count, rate |
| FE-06 | Major | Lint is clean | 0 errors and 0 warnings with eslint:recommended (and typescript-eslint recommended for TS); every disable comment has a reason | ESLint | Output |
| FE-07 | Minor | Formatting is clean | 0 files Prettier would change | prettier --check over JS, TS, CSS and HTML | Files |
| FE-08 | Major | One API client | 0 fetch, XMLHttpRequest or axios calls outside the API module, held by a lint rule | Scan; read the lint config | Count, file:line |
| FE-09 | Major | Network errors reach the user | The API client checks status and content type and handles an expired sign-in; 0 empty catches or .catch(() => {}) on requests | Read the client; scan call sites | Count |
| FE-10 | Major | HTML is built safely | 0 innerHTML, outerHTML, insertAdjacentHTML or document.write with non-constant strings outside the one escaping template helper | ESLint no-unsanitized at 0; scan | Count |
| FE-11 | Major | One page shell | Header, navigation and footer defined once; 0 pages carrying their own copy | Hash the shell markup per page | Pages with copies |
| FE-12 | Major | Shared UI helpers once | Dialog, notification, confirm and formatting (dates, numbers, money) each in one module; 0 same-named function definitions across files outside class methods | Syntax-tree inventory of function names across scripts | Duplicates |
| FE-13 | Minor | No native blocking dialogs | 0 alert, confirm or prompt calls | Scan | Count |
| FE-14 | Minor | Modules are a readable size | 0 modules over 800 lines; 0 functions over 80 lines | Line counts; ESLint max-lines-per-function | Lists |
| FE-15 | Minor | Third-party code is managed | Libraries come from the package manager with a lock; 0 hand-copied files without version and licence; SRI on any CDN script | Read package files and the static folder | Files |
| FE-16 | Major | Shared modules are tested | 100% of shared front-end modules imported by at least one unit test | Map test imports to modules | Untested modules |
| FE-17 | Minor | Pages run without errors | 0 console errors and 0 unhandled rejections across the browser test run | Collect console and pageerror events in the run | Count, pages |
| FE-18 | Minor | No dead front-end code | 0 unused exports or files | knip or ts-prune | List |
| FE-19 | Minor | API paths come from one place | 0 API path literals outside the API module or a generated client | Scan for path prefixes | Count |

### Section 6: CSS, design system and accessibility (10 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| CSS-01 | Major | Tokens defined once | Color, type scale, spacing, radius, shadow, z-index and breakpoints exist as tokens in one file; each token defined once per theme | Parse custom property definitions across all stylesheets | Duplicated definitions |
| CSS-02 | Major | No literal colors | 0 color literals outside token definitions | stylelint declaration-strict-value or color-no-hex outside the token file | Count |
| CSS-03 | Major | No literal sizes | 0 literal font sizes, spacing (margin, padding, gap) or radii outside tokens, apart from 0 | stylelint declaration-strict-value for those properties | Count per property |
| CSS-04 | Major | No undefined tokens | 0 var() references to undefined custom properties without a fallback | Parse definitions and references | List |
| CSS-05 | Minor | Few unused tokens | At most 5% of tokens unused | Same parse | Percentage, list |
| CSS-06 | Critical | No inline styles | 0 style attributes in pages and templates; 0 style blocks in pages; 0 style attributes in HTML built in code | HTML parser and code scan | Count, file:line |
| CSS-07 | Major | No CSS written in scripts | 0 element.style assignments and 0 CSS strings in scripts, apart from setting custom properties for dynamic values | Scan | Count |
| CSS-08 | Major | One definition per component | Buttons, dialogs, tabs, cards, form controls, badges and tables each have one base class with modifiers | Selector inventory grouped by component; count separate families | Families per component |
| CSS-09 | Minor | No duplicate selectors | 0 selectors defined in more than one rule block across all stylesheets | stylelint no-duplicate-selectors plus a cross-file script | Count |
| CSS-10 | Minor | No dead selectors | 0 selectors that match no class or id used in any template or script | PurgeCSS rejected list or a script | Count |
| CSS-11 | Minor | No !important | 0 outside a declared utility layer and reduced-motion rules | Scan | Count |
| CSS-12 | Minor | One naming method | 95% or more of class names match the chosen pattern | stylelint selector-class-pattern | Percentage |
| CSS-13 | Major | CSS lint is clean | 0 errors over every stylesheet of every application | stylelint with stylelint-config-standard | Output |
| CSS-14 | Major | A living style guide | A page showing every token and every CSS-08 component with its variants, drawn from the real stylesheet | Compare the guide with the token and component inventory | Missing items |
| CSS-15 | Major | Themes are complete | 0 color tokens missing a value in any theme | Parse each theme block | Missing tokens |
| CSS-16 | Critical | Contrast meets WCAG 2.2 AA | 4.5:1 for normal text, 3:1 for large text and UI components (1.4.3, 1.4.11), for every text and background pair in use, in every theme, including any brand color the product lets a user set | Compute ratios for every token pair in use; axe color-contrast on every page in each theme; test the brand color path with light and dark extremes | Failing pairs |
| CSS-17 | Critical | Automated accessibility is clean | 0 axe violations of impact critical, serious or moderate on every page and main state, in every theme | axe-core through Playwright over every page | Violations per page |
| CSS-18 | Major | Everything works by keyboard | 0 elements with click handlers that aren't buttons, links or inputs, or lack role, tabindex and a key handler (2.1.1) | Browser script listing click listeners per element; a tab walk of each page | Elements |
| CSS-19 | Major | Focus is visible | Every focusable element shows a focus indicator at 3:1 or more against its surroundings (2.4.7, 1.4.11) | Browser script focusing each element and measuring | Failures |
| CSS-20 | Major | Controls have names | 0 controls labelled only by a placeholder or not at all (1.3.1, 3.3.2, 4.1.2) | axe label rules plus a script | Controls |
| CSS-21 | Minor | Pages have structure | Exactly one h1 and a main landmark per page; no skipped heading levels (1.3.1, 2.4.6) | Browser script per page | Pages failing |
| CSS-22 | Minor | Motion respects the setting | 0 animations or transitions over 200 ms not disabled under prefers-reduced-motion | Parse stylesheets | Rules |
| CSS-23 | Minor | Targets are big enough | 0 target-size failures (2.5.8) | axe target-size rule | Count |

### Section 7: tests, tooling and operations (15 percent)

| ID | Sev | What must be true | Threshold | How to measure | Evidence |
|---|---|---|---|---|---|
| TO-01 | Critical | Tests run on what ships | The gate builds or checks its environment from the shipped locks and refuses any other interpreter or package set; 0 differences between the test environment and the image | Compare versions with the lock and the image; start the gate on another interpreter and watch it refuse | Differences, refusal output |
| TO-02 | Critical | The suite passes | 0 failures, 0 errors, 0 skips without a declared reason; a missing tool fails, never skips | Full suite on the shipped stack | Totals, skip list |
| TO-03 | Major | Tests pass on the shipped OS | The suite (or every OS-sensitive test) passes inside the built image or its base | Run it in the container | Totals |
| TO-04 | Major | Coverage is high and enforced | Branch coverage of 85% or more per application, held by a floor in the gate; 0 product modules under 70% | Coverage with branch measurement | Percentages, modules under |
| TO-05 | Major | The test pyramid holds | Unit tests are 50% or more of all tests | Classify by marker or folder (unit: no network, no database file, no browser) | Counts per level |
| TO-06 | Major | Every route is tested | 100% of operations and page routes referenced by at least one test | Route inventory against test requests | Untested routes |
| TO-07 | Major | Every page has a browser test | 100% of pages have one or more end-to-end browser tests of their main flow | Page list against browser tests | Pages without |
| TO-08 | Major | Tests are stable | 3 consecutive full runs give identical outcomes | Run 3 times | Differences |
| TO-09 | Minor | Tests test behavior | 0 tests that read source files as text to assert on them; 0 tests without an assertion; 0 tests that depend on local machine state (an existing image, a file outside the repository) | Syntax-tree scan of tests; run in a clean container | Lists |
| TO-10 | Major | Lint is clean with no baseline | 0 findings with a declared rule set that includes errors, bugbear, security, blind except and async rules; no baseline file | ruff (or the slot's linter) | Output |
| TO-11 | Major | Formatting is clean | 0 files the formatter would change | ruff format --check or the slot's formatter | Files |
| TO-12 | Major | Types are strict everywhere | Strict mode over 100% of product modules; 0 errors; 0 per-module relaxations | mypy --strict or pyright strict; read the overrides | Errors, overrides |
| TO-13 | Minor | Functions are annotated | 100% of product functions with typed parameters and return | Syntax-tree scan | Percentage |
| TO-14 | Minor | Public code is documented | 80% or more of public modules, classes and functions documented (PEP 257, JSDoc) | interrogate or ruff D; a JSDoc scan | Percentages |
| TO-15 | Critical | Releases go through one binding gate | One command runs every automated check; the only release path refuses a dirty tree, runs that command on the shipped stack, builds from the committed tree and labels the image with the commit; 0 documented build paths that skip it | Read every documented build path (README, docs, scripts, agent skills); try a release from a dirty tree | Paths found, refusal |
| TO-16 | Major | The gate runs on every change | The full gate runs on every push to the main branch outside the developer's machine (CI, a free self-hosted runner, or a server-side hook) | Read the config; check the last 20 commits have a recorded result | Commits with results |
| TO-17 | Major | Dependencies are locked | Every application has a hash-locked lock file in sync with its manifest | uv lock --check, pip-compile --dry-run, npm ci | Output |
| TO-18 | Major | Dependencies are rescanned | The dependency audit runs in the gate and on a schedule against deployed images, at least weekly | Read the gate and the schedule | Schedule, last run |
| TO-19 | Major | The image is reproducible and lean | Base pinned by digest; no unpinned upgrades; multi-stage; non-root; a HEALTHCHECK; 0 hadolint errors or warnings | hadolint; read the Dockerfile | Output |
| TO-20 | Major | The image boots before it ships | The gate starts the built image, waits for health and checks the reported version and commit, before push | Read the release script; run it to the boot step locally | Output |
| TO-21 | Major | Rollback is ready | The previous release image is kept; deploys use immutable tags or digests; a documented rollback command | Registry listing; deploy config | Tags kept |
| TO-22 | Major | Logs are structured | Logs go to stdout as JSON with time, level, logger and request id; 0 print calls in product code | Run the app, parse every log line as JSON; scan for print | Lines failing, prints |
| TO-23 | Major | Errors are tracked | An error tracker or alert destination set in every deployed environment's configuration | Read provisioning and environment templates | Environments missing it |
| TO-24 | Minor | Health and metrics exist | Separate liveness and readiness endpoints, readiness checks the database; request rate, error rate, latency and job queue length exposed or pushed | Probe; read config | Missing items |
| TO-25 | Major | Upgrades are tested | A test migrates a database from the previous release's schema to the current one, after a backup | Read and run the test | Test name, result |
| TO-26 | Minor | Restore is tested | A test restores a backup into a throwaway instance, boots it and compares row counts | Read and run the test | Test name, result |
| TO-27 | Major | A new developer can start | A fresh clone, following only the README in a clean container, reaches a passing quick test run | Do it | Steps, result |
| TO-28 | Minor | Suppressions are rare and explained | At most 1 inline suppression (noqa, eslint-disable, type: ignore, ts-ignore) per 1,000 lines; 100% with an error code and reason | Scan | Rate, unexplained |
| TO-29 | Minor | Images are signed | Release images signed, and the signature checked at deploy | cosign verify; deploy config | Output |

## Change log

- 2.1, RU-14 (Few magic strings). Old rule: every string literal of 4 or more characters that isn't a docstring, log or user message counted, dict keys and the entries of module-level constant tables included. New rule: dict keys (dict literal keys, subscript indexes, the key given to get, pop or setdefault) and the entries of a module-level list, tuple, set or dict assigned to an UPPER_CASE name are left out; the limit of 10 and the 4-module spread are unchanged. Why: a dict key is the name of a field in a record the code reads or writes, not a magic value, and a named module-level table is the named constant the check asks for, so counting them flagged the fix as the fault. External rule it now matches: Pylint and Ruff PLR2004 (magic-value-comparison), which treats a literal as magic only where a value is compared and names a module-level constant as the remedy.
- 2.1, SEC-05 (No injection). Old rule: 0 queries built from non-constant strings, measured with ruff S608 among others. New rule: 0 queries whose text holds a value from input; values are bound as parameters and a table or column name in the text comes only from a fixed allow-list in code, measured by taint rules from a function's input to the query text; ruff S608 is no longer named. Why: a table or column name can't be bound as a parameter, so a repository that writes its own table name or a checked column into an f-string was counted as injection although no input reaches the text, and S608 flags every formatted SQL string whatever its source. External rule it now matches: the OWASP SQL Injection Prevention Cheat Sheet, defense option 3 (allow-list input validation for table and column names) beside option 1 (prepared statements) for values, and Semgrep's taint-mode SQL injection rules, which report only data that flows from a source to the query.
- 2.1, SEC-28 (Errors reveal nothing). Old rule: 0 responses with upstream error text, every 5xx with a reference in the log. New rule: the same, apart from a deliberate pass-through, an administrator's own connection test (an SMTP server's settings, say) that hands back the upstream server's reply about the settings they entered, with a reference. Why: the reply is the result the test exists to show, about a server the administrator configured, so filtering it out leaves them unable to fix their own settings; counting it as a leak flagged the feature, not a fault. External rule it now matches: OWASP ASVS 4.0.3 V7.4.1, which asks for a generic message when an unexpected or security-sensitive error occurs, and the OWASP Error Handling Cheat Sheet, which is about unexpected errors; an expected diagnostic result for an authorized administrator is neither.
- 2.1, API-11 (Responses are filtered). Old rule: 0 response models that pass through undeclared fields, read from the model config. New rule: the same, apart from a deliberate pass-through, a response model that sends the handler's value unchanged so that documenting an answer can never change it, which counts as filtered when the test suite checks every answer it produces against the model and fails on a field the model doesn't declare (proved by a test of the guard and a passing run), and a column added to a stored row is shown not to reach the answer. Why: such a model is built to describe and check the answer rather than rebuild it, and with the guard no field reaches a caller that the description doesn't name, which is the exposure the check is about; counting it by its config alone flagged the design, not a leak. External rule it now matches: OWASP API Security Top 10 2023, API3 (Broken Object Property Level Security), whose prevention asks for schema-based response validation that defines and enforces the data each method returns.
