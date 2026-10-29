# Execution Plan: Name and Address Submission

> **Date:** 2026-10-29
> **Plan type:** Seed
> **Docs branch:** `docs/brief-2026-10-29`
> **Source Brief:** BFT-20 — *Brief: Name and Address Submission (seed)*
> **Brief approval:** Approved in Jira (status: Done), verified 2026-10-29
> **Docs PR:** — (awaiting publication)
> **Total Statements:** 5 (plus scaffold)
> **Wave 1 scope:** Scaffold + visible UI form + log writer prep

---

## What gets generated

This seed plan produces two outputs, in this order:

1. **Project scaffold** — the only code the plan generates
   - .NET 10 + C# 14 + ASP.NET Core minimal API solution (`.slnx`)
   - React 19 + TypeScript + Vite frontend
   - xUnit v3 backend tests + Vitest + Playwright
   - Configured dev mode (Vite proxies `/api` to Kestrel)
   - Build-to-wwwroot wiring for production
   - Lint + format tooling for both sides
   - `.gitignore` entries for build artifacts, logs, temp files
   - README skeleton with run commands (to be completed by S4)

2. **Five ADO Statements** — runway Statements parented to BFT-19, each ~4h, entering the normal brief flow

---

## Scaffold manifest

| Layer | Choice | Status |
|-------|--------|--------|
| **Backend** | .NET 10 (LTS), C# 14, minimal API; `global.json` pins `10.0.x` with `rollForward: latestFeature` | New |
| **Frontend** | React 19, TypeScript (`strict`), Vite; Node.js 24 LTS (`.nvmrc`) | New |
| **Backend tests** | xUnit v3 with `WebApplicationFactory`; `coverlet.collector` for coverage | New |
| **Frontend tests** | Vitest + React Testing Library + jsdom + `@vitest/coverage-v8` | New |
| **E2E tests** | Playwright (`@playwright/test`), Chromium only; `webServer` wires API + Vite | New |
| **Lint + format** | .NET: `.editorconfig`, `dotnet format`, `Directory.Build.props` (`Nullable=enable`, `TreatWarningsAsErrors=true`); Web: ESLint + Prettier | New |
| **Solution file** | `.slnx` (modernized format) | New |
| **Dev mode** | `dotnet run` (Kestrel) + `npm run dev` (Vite with `/api` proxy to http://localhost:5000) | New |
| **Build+serve** | `npm run build` outputs to `src/Bridgenext.Forge.NameSubmission.Api/wwwroot`; API serves static files + `MapFallbackToFile` | New |
| **CI** | **Not included** — Open Question 3; scaffold does not presume it | — |

**Excludes by design:**
- No feature code or domain business logic
- No database or ORM
- No Dockerfile or container config
- No deployment target
- Placeholder screens named as placeholders, navigation wired but not impl.

**Repository structure** (after scaffold):
```
/
├── Bridgenext.Forge.NameSubmission.slnx
├── global.json
├── Directory.Build.props
├── .editorconfig
├── .nvmrc
├── .gitignore (logs/, bin/, obj/, node_modules/, playwright output)
├── src/
│   ├── Bridgenext.Forge.NameSubmission.Api/
│   │   ├── Program.cs (app startup, Vite proxy wiring)
│   │   ├── appsettings.json
│   │   ├── launchSettings.json (localhost:5000)
│   │   ├── logs/ (runtime only; git-ignored)
│   │   └── wwwroot/ (SPA build output)
│   └── web/ (React app)
│       ├── src/
│       │   ├── App.tsx (placeholder page)
│       │   ├── index.css (reset + global styles)
│       │   └── App.test.tsx (trivial test)
│       ├── vite.config.ts (with /api → API proxy)
│       ├── tsconfig.json (strict)
│       ├── .eslintrc.json
│       ├── prettier.config.json
│       ├── package.json (React 19, TypeScript, Vite, testing deps)
│       └── package-lock.json (committed)
├── tests/
│   ├── Bridgenext.Forge.NameSubmission.Api.Tests/
│   │   ├── Program_Tests.cs (trivial test)
│   │   └── .csproj (xUnit v3, WebApplicationFactory, coverlet)
│   └── e2e/ (own package.json)
│       ├── tests/
│       │   └── example.spec.ts (trivial Playwright test)
│       ├── playwright.config.ts (webServer wires API + Vite)
│       ├── package.json (@playwright/test)
│       └── package-lock.json
└── docs/
    ├── briefs/ (seed brief)
    ├── plans/ (this plan)
    └── reference/
```

**Verification checklist** (before raising scaffold PR):
- [ ] Solution builds: `dotnet build`
- [ ] Web builds: `npm run build` (in web/) creates wwwroot/
- [ ] API runs: `dotnet run` starts on localhost:5000
- [ ] Frontend dev server: `npm run dev` (in web/) starts on localhost:5173, proxies /api
- [ ] Backend tests run: `dotnet test`
- [ ] Frontend tests run: `npm test` (in web/)
- [ ] E2E tests run (headless): `npm test` (in tests/e2e/)
- [ ] Lint passes: `dotnet format --verify-no-changes` + `npm run lint` (in web/)
- [ ] README exists with run commands and setup steps

---

## Statements — the runway

Five Statements, ordered for **visible UI first**, dependency graph below.

| # | Statement | Kind | Scope | Depends on | Can parallel | Wave | Notes |
|---|-----------|------|-------|------------|--------------|------|-------|
| S1 | Submission form UI and client validation | Capability | Build the form component with Name/Address fields, all five screen states (Empty, Loading, Partial, Error, Success), client-side validation, accessibility. Stub the submit call. | Scaffold | S2 | 1 | **Visible in wave 1.** Mocked submit responses. |
| S2 | JSON Lines submission log writer | Enabling | Implement `ISubmissionLog` and `JsonlSubmissionLog`: append-only writes with `FileMode.Append`, `FileShare.Read`, SemaphoreSlim concurrency lock, UTC timestamps, directory creation. Unit tests against temporary paths. | Scaffold | S1 | 1 | Hidden from user, enables S3. |
| S3 | API endpoint: basic submission acceptance and recording | Capability | Implement `POST /api/submissions`, hook to S2's log writer, return `201` with the logged timestamp on success. Minimal error handling; stub error contract for now. | S1, S2 | — | 2 | Visible: real submissions work. Depends on both S1 (to know the shape) and S2 (to write). |
| S4 | API error handling and robustness | Enabling | Complete the error contract: 400/413/415 for invalid input, shape conformance in every environment (no ProblemDetails, no dev exception page), I/O failure as 500. Integration tests assert the full contract. | S3 | — | 2 | Unifies S3; runs in parallel if split. |
| S5 | End-to-end test + complete README | Capability | Playwright test: form → submit → read log file (using temp path). Complete README: prerequisites, stack, dev/prod run steps, test commands, API contract, config reference. Sign off the Epic completion report per BFT-19 §8. | S3, S4 | — | 3 | **Closes the Epic.** Tests the full vertical slice. |

---

## Dependency graph

```
Scaffold
  ├─ S1 (form)          ──┐
  │                       ├─→ S3 (accept + record)  ──┐
  └─ S2 (log writer)    ──┘                          ├─→ S5 (E2E + docs)
                                    S4 (error handling) ──┘
```

**Wave ordering:**

| Wave | Statements | Why | Expected | Parallel cap |
|------|-----------|-----|----------|---|
| **Precondition** | Scaffold | Unblocks all Statements; builds, runs, lints green | 1–2 days | 1 (serial) |
| **Wave 1** | S1, S2 | Visible UI first; no cross-dependencies | ~8h each (2 days) | 2 (parallel) |
| **Wave 2** | S3, S4 | Both gate S5; independent | 3–4h each (1 day) | 2 (parallel) — touch different files |
| **Wave 3** | S5 | Closes the Epic | ~4h (half day) | 1 (consolidation) |

**Total estimate:** 10–12 days of solo effort; ~6–7 days with 2-agent parallel (S1+S2 together, then S3+S4, then S5).

---

## Project knowledge

No project-layer updates proposed. The seed brief's Architect section carries all the decisions; downstream briefs reference it.

---

## Reference material

None. BFT-19 has no attachments; the seed brief captured its description in full.

---

## Execution checklist

**Before publishing:**
- [ ] This plan is written to `docs/plans/2026-10-29--name-and-address-submission--v1.md` on `docs/brief-2026-10-29`
- [ ] Scaffold manifest is concrete (stack, versions, scripts, config listed)
- [ ] Five Statements proposed with clear scope and dependencies
- [ ] Wave ordering justified (UI first, dependency-driven)
- [ ] Open questions listed (none blocking execution)

**Publishing (user confirms):**
- [ ] Commit and push the plan to the docs branch
- [ ] Raise a docs PR (linked to BFT-20, auto-complete off)
- [ ] Comment on BFT-20 with the PR URL
- [ ] Mark: **in review, not approved**

**After docs PR merges:**
- [ ] Create the five Statements in ADO (parented to BFT-19, Related-linked to BFT-20)
- [ ] Generate the scaffold on a feature branch (`feature/BFT-20_scaffold`)
- [ ] Raise scaffold PR (linked to BFT-20, auto-complete off)

**No feature code, no further Statements, no `/forge:test-plan` — only the scaffold and the Statements that will later be briefed.**

---

## Open questions that block execution

**None.** The brief's three open questions are all non-blocking:
- Open Question 1 (Name 100 / Address 500 limits): defaults stand; downstream briefs adopt.
- Open Question 2 (log retention): deferred (no hosted deployment).
- Open Question 3 (CI workflow): not included; scaffold does not presume it.

All are resolvable during review without blocking the plan.

---

## Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
