# Brief: Name and Address Submission (seed)

> **Date:** 2026-10-29
> **Brief type:** Seed
> **Docs branch:** `docs/brief-2026-10-29`
> **Branch:** —
> **Brief path:** `docs/briefs/2026-10-29--name-and-address-submission--v1.md`
> **Source Epic:** BFT-19 — *Name and Address Form with API and File Logging*
> **ADO Brief:** BFT-20
> **ADO Brief URL:** https://bridgenext-engineering-poc.atlassian.net/browse/BFT-20
> **Reference material:** — (BFT-19 has no attachments, checked 2026-10-29; its description is the sole source and is captured below)

---

## Product Owner

### Goal
Give a visitor one simple web page where they enter their **Name** and **Address** and submit them. The input is validated in the browser and again on the server. Each accepted submission is recorded durably as one line in a JSON Lines log file, stamped with a server-generated UTC timestamp. The visitor always learns the outcome: a confirmation on success, or a clear message on failure with their input still in place.

### Why it matters
BFT-19 is an agentic-development exercise. The application is small, so doing it *properly* is the point: separate UI, API, validation and logging responsibilities; validation on both sides; safe file handling; the tests listed in Epic §6; documented run steps. BFT-19 §8 also sets the reporting bar: **nothing is claimed as verified unless it was actually tested.**

### Success definition: the Epic is done when
- Every acceptance criterion in BFT-19 §7 holds (restated below under *Business Analyst*).
- The BFT-19 §6 tests, including one end-to-end test from form to log file, pass on a clean checkout, or any failures are documented.
- A new developer can install, run the app, and run every test suite from the README.
- Each downstream brief closes with the BFT-19 §8 completion report.

### Runway boundary: in this Epic
- One responsive page with Name, Address and a Submit button.
- `POST /api/submissions` (REST, JSON), with server-side validation and the BFT-19 §4 error contract.
- An append-only JSON Lines submission log at `logs/submissions.log` (path configurable), with a server UTC timestamp on every entry.
- Client- and server-side validation with length limits.
- Success, validation, network and server-error feedback; Submit disabled while in flight; values kept on failure.
- The BFT-19 §6 tests, and the README.
- The project scaffold. The seed plan generates it; it is not a Statement.

### Deferred: not in this Epic
- Reading, listing, editing or deleting submissions (no read API, no admin screen).
- Authentication and authorisation. The form is anonymous.
- Any store other than the log file.
- Log rotation, retention or purging of personal data (Open Question 2).
- Rate limiting, bot protection, CAPTCHA (see *Architect → Risks*).
- Hosted deployment, containers, CI/CD (Open Question 3 covers CI).
- Localisation, and structured addresses. Address stays one free-text field, per the Epic.
- The Node.js/Express backend option. BFT-19 says to build one backend only, and .NET 10 was chosen.

---

## Business Analyst

### User stories (capability level)

As a **visitor to the submission page**,
I want **to enter my name and address and submit them**,
So that **my details are recorded, and I know straight away whether that worked**.

As a **maintainer of the application**,
I want **every accepted submission appended to a log file with a server-generated timestamp**,
So that **there is a durable, inspectable, append-only record of what was submitted and when**.

### Acceptance criteria: what "Epic done" means
These are initiative-level criteria, traced to BFT-19. Per-screen criteria belong to the downstream briefs.

1. **Entry** — A visitor can enter a Name and an Address on a single responsive page and submit them. *(§3.1, §7)*
2. **Required fields** — When either field is empty or whitespace-only, submission is refused both in the browser and by the API. *(§3.2, §7)*
3. **Length limits** — Browser and server enforce the same maximum lengths, and over-length input is refused with a validation message. *(§3.3)*
4. **Submission** — Submit sends `{ "name", "address" }` as JSON to `POST /api/submissions`. *(§3.4, §7)*
5. **Server validation** — The API validates every request without trusting the client. Invalid fields, malformed JSON, unsupported content types and oversized bodies each get the appropriate 4xx status and the contract error body. *(§3.5, §4, §7)*
6. **Recording** — For each valid submission the API generates a UTC timestamp and appends one JSON Lines entry (Name, Address, timestamp) to the submission log. Existing entries are never modified or lost. *(§3.5–§3.6, §7)*
7. **Created only once recorded** — `201 Created`, with the same `submittedAt` that was logged, is returned only after the log write succeeds. A write failure returns a 5xx with the contract error body. *(§3.7)*
8. **Confirmation** — The visitor sees a success confirmation after a successful submission. *(§3.8, §7)*
9. **Failure feedback** — The visitor sees a specific, plain-language message for a validation error, a network failure and a server error. No stack trace, exception text or internal path ever reaches the visitor. *(§3.9, §5, §7)*
10. **In-flight behaviour** — While a request is in flight, Submit is disabled and no second submission can be sent by click or by the Enter key. On failure, entered values are kept. *(§3.10)*
11. **Tests** — Automated tests cover every item in BFT-19 §6, use isolated temporary log files, and pass, or any failures are documented. *(§6, §7)*
12. **Documentation** — The README covers setup, configuration, API usage, and the commands to run the app and every test suite. *(§5, §7)*

### Edge cases and business rules
- **Trimming.** Leading and trailing whitespace is trimmed before validation, and the trimmed values are recorded. A field that is empty after trimming counts as missing. Inner whitespace is kept.
- **Length limits** apply to the trimmed value. Proposed: **Name 1–100, Address 1–500 characters** (Open Question 1). JavaScript `.length` and C# `.Length` both count UTF-16 units, so browser and server agree on every input.
- **Characters.** No character-set restriction: apostrophes, hyphens, accents and non-Latin scripts are all valid. Control characters are refused, except line breaks in Address, which is multi-line. JSON serialisation escapes line breaks, so each log entry stays on one line.
- **Request shape.** A missing, `null` or non-string `name` or `address` is a 400. Unknown extra properties are ignored and never recorded.
- **Timestamps** are server-generated, and any client-sent timestamp is ignored. Format: ISO 8601 UTC with milliseconds and `Z`, e.g. `2026-10-28T14:30:00.000Z`. The log entry and the response's `submittedAt` carry the identical value.
- **No de-duplication.** Identical submissions become separate entries. Double-submits are prevented in the UI (AC 10).
- **Log file.** If the file or its directory is missing, the first write creates it. The file is only ever appended to, never truncated.
- **Concurrency.** Simultaneous submissions each produce exactly one complete line, and lines never interleave.
- **Personal data.** Name and Address are personal data. The submission log is the only place they are recorded; diagnostic logs never contain them.

### Surfaces this initiative touches

| Surface | Kind | Notes |
|---|---|---|
| Submission page (`/`) | Web page | The only UI surface: form, inline validation, confirmation, error banner |
| `POST /api/submissions` | REST endpoint | The only API surface |
| `logs/submissions.log` | JSON Lines file | Server-side only; never served; git-ignored |
| `README.md` | Documentation | Setup, configuration, API usage, commands |

### Screen states
Not specified at seed level. The five states for the submission page belong to the downstream brief that owns that page; *UX Designer* below sets the direction those states must follow.

---

## Architect

### Chosen design

#### Topology

```
Browser ── same origin ──► ASP.NET Core on .NET 10 (one process)
  React SPA                  ├─ static files: built SPA in wwwroot
                             └─ POST /api/submissions
                                   ├─ SubmissionValidator   (pure)
                                   ├─ TimeProvider          (UTC timestamp)
                                   └─ ISubmissionLog → JsonlSubmissionLog ──► logs/submissions.log
                                                                              (content root, never wwwroot)
Development: Vite dev server proxies /api → Kestrel. The browser sees one origin, so no CORS.
```

#### Stack (settled; the seed plan generates the scaffold from this)

| Concern | Choice |
|---|---|
| Backend | .NET 10 (LTS), C# 14, ASP.NET Core **minimal API**; SDK pinned in `global.json` (`10.0.x`, `rollForward: latestFeature`) |
| Backend tests | xUnit v3; `Microsoft.AspNetCore.Mvc.Testing` (`WebApplicationFactory`); built-in `Assert`; `coverlet.collector` |
| Frontend | React 19 + TypeScript (`strict`), Vite; Node.js 24 LTS (`.nvmrc` + `engines`); npm with a committed lockfile that pins exact versions at scaffold time |
| Frontend tests | Vitest, React Testing Library, `@testing-library/user-event`, jsdom, `@vitest/coverage-v8` |
| End-to-end tests | Playwright (`@playwright/test`), Chromium only |
| Lint and format | .NET: `.editorconfig`, `dotnet format`, and `Directory.Build.props` with `Nullable=enable` and `TreatWarningsAsErrors=true`. Web: ESLint flat config (`typescript-eslint`, `react-hooks`, `jsx-a11y`) and Prettier |
| UI | Plain CSS Modules. No component, form, schema or HTTP-client library: plain React state, a hand-written validation module, `fetch` |
| Solution | `.slnx` |

#### Repository layout

```
/
├── Bridgenext.Forge.NameSubmission.slnx
├── global.json · Directory.Build.props · .editorconfig · .nvmrc
├── src/
│   ├── Bridgenext.Forge.NameSubmission.Api/     minimal API; serves the built SPA from wwwroot
│   │   └── logs/                                runtime only; git-ignored
│   └── web/                                     React + TypeScript (Vite); tests colocated *.test.ts(x)
├── tests/
│   ├── Bridgenext.Forge.NameSubmission.Api.Tests/   xUnit: unit and integration
│   └── e2e/                                          Playwright (own package.json)
└── docs/
```

#### Responsibilities (BFT-19 §5)

| Unit | Side | Owns |
|---|---|---|
| Submission form component | Web | Fields, field errors, the five states, focus and announcements. Never calls `fetch` directly |
| `validation` module | Web | Pure rule functions and the length constants |
| `submissions` API module | Web | `fetch` with a 15 s `AbortController` timeout. Maps every outcome to a typed result (`success`, `validation`, `server`, `network`) instead of throwing |
| Endpoint | API | Parse → validate → timestamp → record → respond. Thin: no rules, no file I/O |
| `SubmissionValidator` | API | Trimming and every rule. Pure and unit-tested |
| `ISubmissionLog` / `JsonlSubmissionLog` | API | The only component that touches the file system |
| Error shaping | API | Maps *every* failure (validation, malformed JSON, 413, 415, I/O, unhandled) to the contract error body in **every environment, Development included**. No ProblemDetails, developer exception page, exception text or paths on `/api/*` |

#### API contract (BFT-19 §4, made exact)
`POST /api/submissions`, `Content-Type: application/json`, body `{ "name": string, "address": string }`, camelCase.

| Status | When | Body |
|---|---|---|
| `201 Created` | Valid, and the log write succeeded | `{ "success": true, "message": "Submission saved successfully.", "submittedAt": "2026-10-28T14:30:00.000Z" }` |
| `400 Bad Request` | Malformed JSON; missing, `null` or non-string field; empty or whitespace-only value; over-length value; control character | `{ "success": false, "message": "<plain reason, e.g. Name and address are required.>" }` |
| `413 Payload Too Large` | Body over **16 KB** | Contract error body |
| `415 Unsupported Media Type` | Content type is not `application/json` | Contract error body |
| `500 Internal Server Error` | Log write failed, or an unhandled error | `{ "success": false, "message": "Your submission could not be saved. Please try again." }` |

There is no `Location` header on `201`, because there is no resource to fetch. 16 KB is about 4× the worst case: 600 maximum-length characters, each escaped as six bytes, plus JSON overhead.

#### Submission log
- **Location:** configuration key `SubmissionLog:FilePath` (environment variable `SubmissionLog__FilePath`), default `logs/submissions.log`, resolved against the API's content root and never under `wwwroot`.
- **Format:** UTF-8 without BOM, `\n` line endings, one `System.Text.Json` object per line, camelCase like the API:
  `{"name":"Alex Johnson","address":"123 Example Street, Hyderabad","submittedAt":"2026-10-28T14:30:00.000Z"}`
  The default encoder escapes non-ASCII as `\uXXXX`. The output is valid JSON either way; the downstream log brief decides whether to relax that for readability.
- **Append:** create the directory if missing; open with `FileMode.Append` and `FileShare.Read`; serialise the whole line in memory, write it in one call, and flush before returning `201`.
- **Concurrency:** a singleton writer serialises writes with `SemaphoreSlim(1, 1)`. The app runs as one instance, so writers in other processes are unsupported, and `FileShare.Read` makes one fail loudly (500) rather than interleave.
- **Failure:** an I/O or permission error means no `201`. The client gets the contract `500`; the server logs the exception type and trace id, never the Name or Address.

#### Hosting and run modes
- **Development:** `dotnet run` starts Kestrel on a fixed port (`launchSettings.json`); `npm run dev` starts Vite with `server.proxy` forwarding `/api` to it.
- **Built:** `npm run build` outputs into the API's `wwwroot`, which serves the static files plus `MapFallbackToFile("index.html")`, so the app runs as one process.
- No container and no deployment target in this Epic.
- Security controls are listed in *Technical Lead → Security checklist*.

### Alternatives considered and ruled out

| Alternative | Reason rejected |
|---|---|
| Node.js + Express (BFT-19 option 2) | The product owner chose .NET 10 on 2026-10-29: the repo name follows .NET convention and the repo is worked in Visual Studio 2026. The Epic forbids building both |
| MVC controllers | One endpoint. Controllers add ceremony, and their ProblemDetails pipeline would have to be overridden to keep the contract shape |
| Built-in minimal-API validation / DataAnnotations | Can't cleanly express "whitespace-only is missing, record the trimmed value", and it returns validation-problem bodies rather than the BFT-19 shape |
| Serilog or NLog file sink as the submission log | The submission log is business data, not diagnostics. A logging framework adds envelope fields, buffering and rolling behaviour we don't control |
| SQLite or another database | BFT-19 mandates a JSON Lines file and no unnecessary infrastructure |
| Separate frontend origin with CORS | Two origins to configure and secure, for no benefit at this size |
| Visual Studio "React and ASP.NET Core" template (SpaProxy + `.esproj`) | Adds `Microsoft.AspNetCore.SpaProxy` and Visual Studio-only project tooling. The Vite proxy does the same job with plain CLI commands |
| Form or schema libraries (React Hook Form, Zod) | Two fields don't earn the dependency |
| Jest / Cypress | Vitest reuses Vite's config. Playwright's `webServer` starts both the API and Vite with less setup |

### Risks and dependencies
- **Client/server validation drift.** The limits live in two languages. Mitigation: identically named constants, both suites test the same boundaries (100/101, 500/501), and the server is authoritative.
- **Framework defaults leaking the wrong error shape.** Kestrel's 413, the binding 415, malformed-JSON failures and the developer exception page bypass the contract body unless error shaping covers each. Integration tests must assert the **body**, not just the status, for every contract row.
- **Torn or interleaved log lines.** Mitigated by the single serialised writer and one-call writes. Running more than one instance voids the guarantee (out of scope).
- **Anonymous endpoint abuse.** There is no rate limiting, so a script could fill the disk. The 16 KB limit bounds each request. ASP.NET Core's built-in rate limiter is the cheap fix once the app is exposed beyond local use.
- **Personal data accumulates without a retention policy** (Open Question 2). This blocks any hosted deployment, not the runway.
- **Disk-full and permission errors** must surface as the contract `500`. Tests simulate them with an unwritable temporary path.
- **Prerequisites:** the .NET 10 SDK, Node.js 24, and Playwright's Chromium download. The README lists them.

### Inbound dependencies (in flight)

_Verified against `main` @ `87c525d`, 2026-10-29 06:13 UTC; re-checked at approval._

None. Surveyed, and nothing this brief depends on is in flight. Origin has only `main` (README only), there are no open PRs, no BFT issue is parented to or linked with BFT-19, and the repo has no approved plans. BFT-2 / BFT-4 (Leave Management Web API) is a separate initiative and shares no capability with this one.

### Schema changes
None; there is no database. The log line format above is the only persisted shape, and changing it later breaks anyone who reads the file.

### API contract changes
New endpoint: `POST /api/submissions`, as above. There is no existing API.

---

## UX Designer

### Experience assessment
- **One page, one task.** A centred single-column card (about 32rem wide at most) with a heading, two labelled fields and one primary button. No navigation.
- **Fields.** Name: `<input type="text" autocomplete="name">`. Address: `<textarea autocomplete="street-address">`, about three rows. Labels are always visible, never placeholder-only. "Required" is stated in text, not by colour or an asterisk alone.
- **Validation timing.** Validate on submit. After that, re-validate a flagged field as it changes, so the error clears as soon as it is fixed. Never flag a field the visitor hasn't attempted.
- **Client validation failure:** inline error under each invalid field, linked with `aria-describedby`; `aria-invalid="true"` on the field; focus moves to the first invalid field.
- **Server validation failure** (only possible through drift): shown in the error banner with the server's message. The server is authoritative.
- **Network or server failure:** a `role="alert"` banner above Submit. Network message: "We couldn't reach the server. Check your connection and try again." Server failures use the contract message. Raw status codes are never shown.
- **First surface for early feedback: the submission page itself**, built against a stubbed submit call, so the full interaction can be reviewed before the API or the log exists.

### State-by-state review (initiative direction; the exact design belongs to the form brief)

| State | UX assessment | Concerns |
|---------|--------------|----------|
| Empty | Blank form, no errors shown, Submit enabled | Don't disable Submit until the form is valid: a disabled button hides *why* nothing happens |
| Loading | Submit disabled and reads "Submitting…"; fields read-only so edits can't race the request | Block Enter-key resubmission too |
| Partial | One field valid, one invalid: inline error on the invalid field only | Wording must match the server's message for the same rule |
| Error | Alert banner; values kept; Submit enabled again for retry | No stack traces, status codes or paths |
| Success | Confirmation in a `role="status"` region; fields cleared | The confirmation is replaced once a new entry starts, so it never sits beside new input |

### Platform considerations (responsive web)
- 320px wide to desktop, single column. Touch targets at least 44 × 44 CSS px. Input font size at least 16px, so iOS doesn't zoom on focus.
- Keyboard: Name → Address → Submit. Enter in Name submits; Enter in Address inserts a line break.
- **WCAG 2.1 AA:** contrast at least 4.5:1; errors never conveyed by colour alone.
- Evergreen Chrome, Edge, Firefox and Safari; end-to-end tests run on Chromium. No offline mode: a network failure is the Error state.

### Concerns for Technical Lead
- Validation messages are written in two places. The form brief should hold one wording table that both sides follow.
- Clearing the form on success is a choice (see *Trade-offs*). If review contests it, settle it in the form brief.
- The `role="status"` region has to be in the DOM before its text changes, or screen readers won't announce it.

---

## Technical Lead

### Work item metadata (Jira)

| Field | Value |
|-------|-------|
| Source Epic | BFT-19, the Brief's parent |
| Source Statements | None (Epic-sourced seed). The seed plan creates runway Statements under BFT-19 (Jira type `Statement`, per CLAUDE.md), each Relates-linked to this Brief |
| Brief | BFT-20, a Jira `Story` under BFT-19 |
| Area Path / Iteration | N/A in Jira / not set |
| Labels | `seed-brief`, `brief-pending` |
| Priority | Not set (Jira default) |
| Estimate | N/A. A seed brief is unestimated, and so are its runway Statements |
| Branch | Scaffold branch, named when the seed plan runs: `feature/BFT-20_scaffold` |
| Feature flag | N/A |

### Capability map (seed brief; this replaces the implementation sequence)

**Precondition: the scaffold.** A seed-plan output, not a Statement. It contains the solution and the API project with no domain endpoints; the Vite React app with a placeholder page; the `/api` proxy and the build-into-`wwwroot` wiring; xUnit, Vitest and Playwright harnesses, each with at most one trivial test; lint and format configuration; `global.json`, `.nvmrc`, `Directory.Build.props`, `.editorconfig`; `.gitignore` entries for `bin/`, `obj/`, `node_modules/`, `logs/` and Playwright output; and a README skeleton with the run commands.

| # | Capability | Kind | Depends on | Unblocks | Visible? |
|---|---|---|---|---|---|
| C1 | **Submission form.** The page, both fields, client-side validation, all five states, accessibility. Submits through the web `submissions` module, which here is a **named stub** returning contract-shaped responses, switchable to simulate each failure | Capability | Scaffold | C3 (replaces the stub) | **Yes, wave 1** |
| C2 | **Submission log.** The JSON Lines writer: timestamp, append, directory creation, concurrency, failure semantics; tested against temporary files. No HTTP | Enabling | Scaffold | C3 | No |
| C3 | **Submissions accepted and recorded by the API.** The endpoint, server validation, the full error contract, `201` only after the write, and C1's stub replaced by the real call | Capability | C1, C2 | C4 | Yes: real submits |
| C4 | **End-to-end verification and docs.** A Playwright test from form to log file (temporary path), the README completed, and the Epic-level completion report | Capability | C3 | closes the Epic | No |

The seed plan splits these into Statements of about half a day each. C3 is the most likely to need splitting (accept-and-record on one side, the error contract and robustness on the other). Expect about four to six Statements: the Epic is small, and the seed exists mainly because the repository is empty.

### Reuse opportunities
- The built-in `TimeProvider` for timestamps, with a fixed provider in tests. No custom clock.
- One camelCase `System.Text.Json` configuration for both request binding and log lines, so log keys always match API keys.
- One `WebApplicationFactory` fixture that overrides `SubmissionLog:FilePath` with a per-test temporary file.
- Playwright `webServer` starts both the API (with `SubmissionLog__FilePath` set to a temporary file) and Vite, and the test reads that file back.

### Telemetry events
There is no analytics platform in scope. These are structured `ILogger` events, and **none carries the Name or the Address.**

| Event | Trigger | Required properties |
|-------|---------|---------------------|
| `submission.recorded` | Write succeeded, `201` returned | `event_id`, `trace_id`, `timestamp`, `source` |
| `submission.rejected` | 400 / 413 / 415 returned | the above + `reason` (the rule or status, never the input) |
| `submission.record-failed` | Write threw, `500` returned | the above + `exception_type` |

### Security checklist
**OWASP Top 10**
- [ ] A01 Broken access control: anonymous by design (BFT-19); the only routes are the SPA and `/api/submissions`; the log file is outside `wwwroot` and never served
- [ ] A02 Cryptographic failures: HTTPS redirection and HSTS outside Development; plaintext personal data in the log is accepted for this Epic, with retention as Open Question 2
- [ ] A03 Injection: server-side validation on every request; log lines built by JSON serialisation, never string concatenation; React escaping, no `dangerouslySetInnerHTML`
- [ ] A04 Insecure design: 16 KB body limit; rate limiting deferred (see *Architect → Risks*)
- [ ] A05 Security misconfiguration: contract-shaped errors in every environment; no CORS; `X-Content-Type-Options: nosniff`. CSRF is N/A because there are no cookies or credentials; revisit if authentication is added
- [ ] A07 Auth failures: N/A, there is no authentication in scope
- [ ] A09 Logging failures: failures logged with a trace id; Name and Address never in diagnostic logs

### Offline / degraded-network behaviour
- [ ] Network unavailable or the 15 s timeout: Error state with a retry message, values kept
- [ ] Cached reads: N/A (no reads)
- [ ] Mutations fail fast with a retry prompt: yes, via the timeout
- [ ] Refetch on reconnect: N/A. No automatic resubmission, to avoid duplicates

### Definition of done (the floor every downstream brief inherits)
- [ ] `dotnet build` and `npm run build` green; `tsc --noEmit` passes
- [ ] `dotnet format --verify-no-changes`, ESLint and the Prettier check pass with no new suppressions
- [ ] Unit tests written; ≥ 85% line coverage on changed files
- [ ] All five screen states implemented for the submission page (or N/A with a reason)
- [ ] The telemetry events above emitted, with no personal data
- [ ] Security checklist complete; no secrets in source; `logs/` git-ignored
- [ ] Tests use isolated temporary log files and never touch `logs/submissions.log`
- [ ] Branch `feature/BFT-<n>_<short-summary>`; PR title `[Verb] [Short title]` ≤ 70 characters, carrying the Jira key
- [ ] The PR description carries the BFT-19 §8 completion report: technologies and architecture, files changed, endpoint and log location, tests run **with actual results**, outstanding issues. Nothing untested is claimed as verified
- [ ] README and the project knowledge layer updated where the brief changes them

### Parallelisation recommendation
After the scaffold, C1 and C2 share no files and can be built in parallel. C3 waits for both, and C4 waits for C3. This needs user approval when the seed plan sets its waves.

---

## Trade-offs
- 2026-10-29 — Backend is .NET 10, not Node.js/Express: chosen by the product owner; the repo is .NET-named and BFT-19 forbids building both. Revisit when: never within this Epic (switching means re-scaffolding).
- 2026-10-29 — Hand-written pure validator: exact control of the error shape and the trimming semantics. Revisit when: several endpoints share rules.
- 2026-10-29 — Trim, and record the trimmed values: surrounding whitespace is noise, and "whitespace-only is missing" needs no special case. Revisit when: a log consumer needs verbatim input.
- 2026-10-29 — Same-origin topology (Vite proxy in development, the API serves the SPA): no CORS surface. Revisit when: the frontend and API need independent deployment.
- 2026-10-29 — In-process writer lock only: the app runs as one instance. Revisit when: more than one process writes the log.
- 2026-10-29 — No `Location` header on `201`: there is nothing to fetch. Revisit when: a read endpoint exists.
- 2026-10-29 — Clear the form on success: it signals completion and prevents accidental resubmission. Revisit when: visitors need to submit near-identical variations.
- 2026-10-29 — Limits of Name 100 and Address 500, counted in UTF-16 units after trimming: generous for real data, and they bound log growth. Revisit when: Open Question 1 is answered.
- 2026-10-29 — No form, schema or HTTP-client libraries, per BFT-19 §5 "avoid unnecessary dependencies". Revisit when: the app gains more forms.
- 2026-10-29 — Seed brief for a small Epic: the repo is empty and needs a scaffold first, so the runway will be short. Revisit when: N/A.

---

## Open Questions

| Question | Owner | Status | Resolution |
|----------|-------|--------|------------|
| 1. Are the maximum lengths of Name 100 and Address 500 characters right? | Product owner (BFT-19 reporter, Kiran Kale) | Open, non-blocking: the defaults stand unless changed | Answer in review; the form and API briefs adopt it |
| 2. What retention, rotation and purge policy applies to `logs/submissions.log`, which holds personal data? | Product owner | Open, non-blocking for this Epic (no hosted deployment) | Decide before any hosted deployment; may become its own Statement |
| 3. Should the scaffold include a CI workflow (GitHub Actions running build, lint and tests)? | Product owner | Open, non-blocking. Default: no CI (BFT-19 §5, "avoid unnecessary infrastructure") | Answer in review; if yes, the seed plan's scaffold includes it |

---

## Test Data
N/A by design — a seed brief ships only the project scaffold, so there is no data to seed. Test data is defined per downstream brief.

---

## Tester
N/A by design — a seed brief ships only the project scaffold, which has no acceptance criteria, so `/forge:test-plan` does not apply. Each downstream brief derived from this one carries its own test plan.

---

## Approval
Status: **Approved**
Date: 2026-10-29
ADO Brief: BFT-20
ADO Brief URL: https://bridgenext-engineering-poc.atlassian.net/browse/BFT-20
