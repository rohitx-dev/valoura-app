# Valoura: phased implementation and solo team workflow

Prepared for Rohit Singh · 11 September 2026

Follow this alongside Valoura-MVP-Complete-Guide.md. That document defines the requirements, API contracts, business rules and tests; this guide defines your execution order and team handoffs. V-01 through V-28 refer to its existing backlog. They are planning IDs, not GitHub issue numbers.

You are one developer taking different roles at different times. Each feature goes through product definition, technical design, implementation, review, testing and release. No repository, issues or application code have been created by this guide.

**Development order:** establish GitHub and project foundations → build the homepage with fixtures → establish data and API foundations → build identity → vendor onboarding and moderation → integrate discovery → availability → requests and quotes → reservation logic → payments → management/refunds/transfers → release.

Build and integrate one useful feature at a time. Do not finish the entire backend before touching the frontend. The homepage starts in Phase 3. For later features, agree the API contract, implement the backend rules, connect the frontend, and verify the complete journey.

## 1. Your simulated delivery team

| Role you take | Responsibility | Handoff evidence |
| --- | --- | --- |
| Product manager (PM) | Choose the next user outcome, scope and priority | Issue with acceptance criteria and dependencies |
| Technical lead (TL) | Decide contracts, data changes, failure handling and task boundaries | Technical checklist or architecture decision |
| UI/UX designer | Define screen sequence, layout and loading/error/empty states | Wireframe or annotated screen requirements |
| Frontend developer (FE) | Build accessible pages and connect APIs | UI code, screenshots and integration evidence |
| Backend developer (BE) | Implement validation, permissions, persistence and business rules | API code, contract updates and targeted tests |
| QA engineer | Try valid, invalid, unauthorized and interrupted journeys | Actual test results and reproducible bug reports |
| Reviewer | Read the diff against the issue and question assumptions | Self-review notes; independent review when a real collaborator exists |
| DevOps/release engineer | Maintain CI, environments, deployment and recovery | Green checks, staging evidence and release record |

Assign GitHub issues to your real account. Add a `Role` field or labels such as `role:backend` to represent the current responsibility. Do not invent other contributors or use fake accounts to approve your work. Customer/vendor/admin are application roles; FE/BE/QA are development responsibilities.

GitHub does not allow authors to approve their own PRs. For solo work, use a written self-review and required automated checks, with zero required external approvals. When a real teammate joins, enable a separate required reviewer. [S2]

## 2. Stack and when to introduce it

These choices preserve the actual architecture document. Earlier informal suggestions are superseded by that baseline unless you deliberately amend it.

| Technology | Introduce | Use |
| --- | --- | --- |
| Git, GitHub Issues, Projects and PRs | Phase 1 | Planning, version history and review |
| Node.js supported LTS, npm workspaces, TypeScript | Phase 2 | Reproducible monorepo foundations |
| Next.js App Router, React, Tailwind | Phase 2 scaffold; Phase 3 UI | Public pages and interactive dashboards |
| Express + TypeScript | Phase 2 scaffold; Phase 4 APIs | One backend containing feature modules |
| ESLint, Prettier, GitHub Actions | Phase 2 | Consistent code and automated checks |
| MongoDB replica set + Mongoose | Phase 4 | Models, indexes and multi-document transactions |
| Zod and OpenAPI | Phase 4, then every API feature | Runtime validation and shared API contract |
| MongoDB-backed opaque sessions, Argon2id | Phase 5 | Browser authentication and password hashing |
| SMTP adapter/local mail catcher | Phase 5 | Verification/reset mail; later notifications |
| Cloudinary upload adapter | Phase 6 | Vendor gallery images; MongoDB stores metadata |
| Vitest + Supertest | Phase 4 onward | Business-rule and real-database API tests |
| Playwright | First integrated journey onward | Customer/vendor/admin browser flows |
| MongoDB jobs/outbox + one worker | Scaffold Phase 2; durable execution Phase 9 | Expiry, retries, notifications and reconciliation |
| Razorpay Orders + Standard Checkout | Phase 10 | Sandbox customer payment |
| Razorpay Route adapter | Phase 12 | Separate vendor transfer/settlement tracking |
| Hosting for web/API/worker and managed database | Staging in Phase 4; release Phase 13 | Early deployment feedback and final release |

Choose a supported Node LTS compatible with your selected Next.js release during setup. Record exact versions in the repository and commit the lockfile; do not repeatedly scaffold with whatever version happens to be latest. Next.js supports TypeScript directly. [S4]

Use the existing document's monorepo paths if already established. Otherwise use `apps/web`, `apps/api`, `packages/contracts`, `docs` and `.github/workflows`; keep the worker in the backend codebase. Browser API calls use the documented same-origin `/api/v1` proxy to Express. Avoid implementing a second independent backend in Next.js route handlers.

Redis, Elasticsearch, microservices and Kubernetes are not prerequisites for this MVP. Add a tool only when a measured requirement justifies it. Learn the current phase's concepts, then immediately apply them.

## 3. Phase 0 — Establish your actual starting point

**Owner:** PM → TL. **Backlog:** V-01. **Outcome:** a reliable starting checklist.

You previously reported completing MongoDB connection and authentication and starting Next.js. Treat those as existing work to inspect, not as a reason to delete the project or assume every requirement passes.

1. Inspect the existing repository and run the existing applications.
2. Record each area as working, partial, missing or needs migration, with evidence.
3. Compare auth against the documented session contract. If existing code uses JWT or JavaScript, create a focused migration issue, or explicitly revise the architecture before continuing. Do not mix two auth schemes accidentally.
4. Check the customer/vendor/admin role names, directory structure and API prefix for mismatches.
5. Preserve working code and Git history. Create follow-up issues for gaps.

**Done when:** `docs/current-state.md` identifies reusable work and the next incomplete prerequisite. A reported feature is accepted only after its relevant acceptance checks pass.

## 4. Phase 1 — Establish GitHub before feature development

**Owner:** PM + TL. **Backlog:** V-01. **Branch:** `docs/mvp-baseline`.

1. Use one Valoura repository. Retain an existing one if it contains the work.
2. Commit the prior documentation plus this workflow under `docs/`.
3. Create milestones M0 Foundation, M1 Discovery/Identity, M2 Vendor operations, M3 Booking, M4 Payments, M5 Management/Money operations, M6 Release.
4. Create a GitHub Project board: Backlog → Ready → In progress → In review → Done. Add a Blocked label and dependency note rather than hiding blocked work.
5. Create the V-01–V-28 issues from the existing backlog, or create each upcoming milestone's issues before starting it. Keep the V-ID in each title and record actual GitHub issue numbers.
6. Add Priority, Area and Role labels/fields. Set work-in-progress to one implementation issue.
7. Add an issue template and `.github/pull_request_template.md` using the templates below.
8. Configure PR-based changes and required CI checks on main where your repository plan supports enforcement. Enable checks after the workflow has run and GitHub can identify their names. Keep force pushes/deletion of main blocked. [S3]

Use GitHub Flow: one shared `main` branch and short-lived task branches. This is a documented professional workflow, not the only workflow every company uses. [S1] You do not need permanent `frontend`, `backend`, or person-named branches. Staging and production are environments; they do not require separate permanent branches.

**Fresh repository only:** create an empty GitHub repository without an initial README, then use Git Bash on Windows:

```bash
mkdir valoura
cd valoura
git init -b main
git config user.name "Rohit Singh"
git config user.email "YOUR_VERIFIED_OR_GITHUB_NOREPLY_EMAIL"
```

Create a README and `.gitignore` before the next commands. Ignore dependency/build outputs and real environment files; allow sanitized `.env.example` files. Then:

```bash
git add README.md .gitignore
git diff --cached
git commit -m "chore: initialize Valoura repository"
git remote add origin https://github.com/YOUR_USERNAME/valoura.git
git push -u origin main
git switch -c docs/mvp-baseline
```

The first push bootstraps main. Subsequent work uses PRs. If GitHub already contains the project, use `git clone` or the existing checkout instead; do not run this initialization sequence inside it. Authenticate using Git Credential Manager or SSH, never a token embedded in a remote URL.

**Done when:** the planning PR is merged, the board exists, and V-02 or its remaining alignment work is Ready.

## 5. Phase 2 — Scaffold runnable applications and CI

**Owner:** TL → BE/FE → DevOps. **Backlog:** V-02 and foundational V-03. **Branches:** `chore/ISSUE-workspace-setup`, then `ci/ISSUE-quality-checks`.

1. Align npm workspaces, one root lockfile and TypeScript settings.
2. Scaffold or align Next.js with App Router, TypeScript and Tailwind.
3. Scaffold Express with configuration validation, API health endpoint, structured errors and request IDs. Scaffold a worker entry point.
4. Add sanitized environment examples and instructions for running web/API/worker. Keep server secrets out of `NEXT_PUBLIC_` variables.
5. Define root scripts for `dev`, `lint`, `typecheck`, `test` and `build`. These names are a project convention to implement, not commands that exist automatically.
6. Add PR CI: `npm ci`, lint, typecheck and build; introduce meaningful tests as business logic appears. Later add replica-set integration and browser jobs.
7. Verify from a clean checkout/install; do not rely on untracked local configuration.

**Done when:** web renders, API health responds, development proxy works, and CI passes. A placeholder home is sufficient here.

## 6. Phase 3 — Build the homepage and public UI with fixtures

**Owner:** PM → UI/UX → FE → QA. **Backlog:** V-05. **Branch:** `feat/ISSUE-homepage-ui` with separate PRs for results/profile if needed.

1. Turn W-01/W-02/W-03 into a component checklist.
2. Build header/footer, typography, buttons, form fields and responsive layout.
3. Build homepage search, categories and vendor cards using fictional fixture data.
4. Build results layout, filters, pagination controls and vendor profile/gallery.
5. Define loading, empty and error states now; isolate fixtures behind a data-access function so integration is straightforward.
6. Check 360 px mobile width, desktop layout and keyboard navigation. Add screenshots to the PR.

**Learn now:** React props/state, TypeScript object types, CSS flex/grid, responsive layout and URL query state.

**Done when:** a visitor can navigate home → results → vendor profile with fixtures. This is a UI milestone, not evidence that discovery APIs or booking work.

## 7. Phase 4 — Data, contracts and an early staging skeleton

**Owner:** TL → BE → QA/DevOps. **Backlog:** remaining V-03 and V-04.

1. Document request/response DTOs, errors, pagination and role rules in OpenAPI/shared contracts.
2. Connect an isolated development MongoDB replica set. Prepare a separate test database with transaction support.
3. Implement foundational user/vendor/category/city schemas and index migration commands. Add booking/payment models with their later feature phases once contracts are ready; never omit their indexes when introduced.
4. Seed at least 12 fictional vendors across photographers, makeup artists and decorators, including unpublished/suspended test cases.
5. Make seed and index setup repeatable. Verify actual indexes, not just schema declarations.
6. Add API integration test infrastructure against the real test database; fake external providers at the adapter boundary.
7. Deploy a thin staging web/API skeleton and database connection early. Record chosen hosting, same-origin routing, environment variables and logs. Introduce the persistent worker deployment when Phase 9 needs it.

**Done when:** a clean environment can initialize safely, API errors match the contract, and staging web/API communication works. Final release hardening remains Phase 13.

## 8. Phase 5 — Complete authentication end to end

**Owner:** TL → BE → FE → QA. **Backlog:** V-06/V-07 or audit-gap issues.

1. Define auth endpoints, cookies, CSRF and role/ownership middleware against the existing contract.
2. Implement or migrate registration, password hashing, login, logout, current-user lookup, session expiry/revocation and rate limiting.
3. Add single-use email verification/password-reset flows through the mail adapter.
4. Connect registration/login/reset UI and role-appropriate navigation.
5. Test unauthorized access by changing resource IDs, attempted admin registration, logout, reset-token reuse and session invalidation. Hiding a button is not backend authorization.

**Done when:** T-01/T-02/T-03/T-34 pass as relevant; authentication works in the browser through staging cookie/proxy settings. Existing working auth is credited after verification.

## 9. Phase 6 — Vendor onboarding, gallery and admin moderation

**Owner:** PM/UI → BE → FE → QA. **Backlog:** V-08/V-09.

1. Implement owned vendor draft profile operations and field validation.
2. Add backend-controlled Cloudinary upload authorization, file constraints and asset ownership checks.
3. Build vendor profile/gallery editing and submission screens.
4. Implement admin approval/rejection/suspension and approved-public-snapshot behavior.
5. Build the moderation queue and record reasons/audit events.
6. Verify foreign asset IDs, invalid uploads, stale approval versions and suspended listings.

**Done when:** a new vendor can submit a profile and an admin can publish it; unapproved changes do not leak into public discovery. Tests T-04/T-05 cover the critical boundaries.

## 10. Phase 7 — Connect discovery and build availability

**Owner:** BE → FE → QA. **Backlog:** V-10/V-12. V-11 favourites is P1 and can wait until after core money operations.

1. Implement approved-only listing/detail endpoints with category, city, price filtering, stable sorting and pagination.
2. Replace fixture data access with real APIs in homepage/results/profile. Keep filter state in the URL.
3. Verify loading/errors/empty results and rejection of access to unpublished profiles.
4. Implement vendor/date blocks and the vendor calendar, respecting the one-vendor-per-day rule.
5. Expose public unavailability without customer booking details. Ensure manual blocks cannot overwrite held/confirmed slots.

**Done when:** the public journey uses real data, filters work after reload, and vendor date blocks are respected. T-06 and availability acceptance criteria pass.

## 11. Phase 8 — Booking requests and quotes

**Owner:** PM/TL → BE → FE → QA. **Backlog:** V-13/V-14.

1. Revisit the documented state transitions and date rules before writing handlers.
2. Implement a verified customer's dated request, duplicate-active-request prevention and own booking detail.
3. Connect the request form, validation messages and requested-state confirmation.
4. Implement vendor quote/decline: fixed all-inclusive integer-paise total, scope and 24-hour validity capped by the event day's start.
5. Build vendor request detail and customer quote summary with explicit status/expiry.
6. Enforce server-side time and ownership checks; a request or quote does not reserve the date.

**Done when:** customer requests → vendor quotes → customer sees the quote, with no real payment yet. T-07/T-08 and quote acceptance criteria pass.

## 12. Phase 9 — Durable jobs and exclusive checkout holds

**Owner:** TL/BE → QA → FE. **Backlog:** V-15/V-16.

1. Implement durable jobs, leases, retries and outbox processing; deploy the worker to staging.
2. Add expiration jobs for requests, quotes and holds, plus checks on request paths so worker delays do not extend eligibility.
3. Use unique vendor/date indexes and transactions to claim the slot for one booking.
4. Apply the 15-minute hold capped by quote expiry, reject checkout with less than 60 seconds remaining, and snapshot accepted scope/price/policy.
5. Create a local payment intent before external payment work. Make retries converge on the intended checkout rather than creating duplicates.
6. Add frontend hold countdown, conflict and expired states using server deadlines.
7. Run simultaneous checkout attempts and worker-restart tests.

**Done when:** T-09/T-10/T-11 pass and two customers cannot both own the same slot. Do not build checkout money movement on top of an unproven reservation mechanism.

## 13. Phase 10 — Sandbox payments and recovery

**Owner:** TL → BE → FE → QA. **Backlog:** V-17/V-18/V-19. Split these into focused PRs.

1. Implement the Razorpay adapter in test mode; create provider orders from server-stored quote amount and INR.
2. Connect hosted checkout and show order-setup pending, payment-verifying, confirmed, failed and expired states.
3. Verify checkout signatures server-side. Handle signed webhooks using the original raw request bytes. [S5/S6]
4. Persist events durably and make duplicate delivery safe; provider events can repeat and arrive out of order. [S6]
5. Use one capture-allocation service for webhook and browser verification paths. Confirm only captured payment matching the order/amount/currency and a currently owned unexpired hold.
6. Reconcile provider timeouts, closed browsers and missing/delayed events. External provider calls are outside MongoDB transactions; durable intents bridge failures between the two systems.
7. Route late/duplicate/unallocatable captured charges into technical refund intents. Never revive an expired booking or take another booking's slot.

**Done when:** T-12 through T-19 plus T-29/T-35 pass as applicable, and staging receives a real sandbox webhook. Refund intent recording is built here; actual refund execution is completed in Phase 11 before releasing the full journey.

## 14. Phase 11 — Dashboards, cancellations and refunds

**Owner:** PM/TL → BE → FE → QA. **Backlog:** V-20/V-21/V-22.

1. Finish customer/vendor booking lists, detail, timeline and payment recovery display; reuse screens introduced in Phase 8.
2. Add cancellation preview using the documented Asia/Kolkata policy boundary, then guarded cancellation and atomic slot release.
3. Add admin refund execution with durable intent, provider reference, retry and reconciliation.
4. Display refund pending/failed/processed using provider evidence; request submission is not completed refund.
5. Test cancellation versus capture races, repeated refund requests, changed preview entitlement and provider timeout after accepting a refund.

**Done when:** T-20 through T-24 pass and an eligible sandbox booking can be cancelled and actually reconciled to a processed refund.

## 15. Phase 12 — Vendor money operations, completion and notifications

**Owner:** BE → FE → QA/Operations. **Backlog:** V-23/V-24/V-25.

1. Store verified provider-linked-account references and implement the transfer adapter.
2. Use approved Route sandbox capabilities where available. Otherwise use an explicitly labelled simulated transfer adapter and keep live payments disabled.
3. Track customer payment, vendor transfer and bank settlement as separate records/states.
4. Handle cancellation during/after a transfer: freeze, reconcile and reverse/recover as applicable while tracking the refund separately.
5. Add completion only after the event day's end, transactional booking/payment/refund emails, and delivery failure visibility.
6. Finish admin exception search, reconciliation actions and audit history. Do not offer arbitrary money-status editing.

**Done when:** T-25 through T-28 pass; a failed transfer does not falsely mark the customer booking unpaid. Admin can identify and act on unresolved money operations. Notification jobs survive worker restarts.

## 16. Phase 13 — Acceptance, staging release and portfolio delivery

**Owner:** QA → DevOps → PM. **Backlog:** V-26/V-27.

1. Run the original acceptance catalog T-01–T-35 at its specified levels; fix failures with bug issues/PRs.
2. Run browser journeys across separate customer/vendor/admin test accounts, including approval, quote/payment and cancellation/refund.
3. Verify mobile, keyboard access and documented performance targets using measured results.
4. Verify production builds, HTTPS/cookies, proxy raw webhook handling, worker recovery and environment separation in staging.
5. Rehearse backup restore and reconcile restored pending money operations.
6. Publish setup instructions, architecture explanation, screenshots, demo accounts where appropriate and known limitations in README. Clearly label test checkout and simulated transfers.
7. Tag the exact tested commit, retain the previous deployable artifact, and deploy that same tested artifact/commit to the intended portfolio environment.

**Done when:** the portfolio checklist in document 08 passes. This is a complete sandbox MVP; it does not imply commercial readiness.

## 17. Phase 14 — Live readiness, only if launching commercially

**Owner:** Product/Operations → DevOps/BE → QA. **Backlog:** V-28.

Complete the previous document's separate live checklist: provider enablement, actual vendor onboarding, operational terms, support responsibilities, refund funding, live secrets isolation, monitoring, and controlled payment/transfer/refund verification. External provider lead time has no reliable coding estimate. Keep the sandbox release usable while these conditions are resolved.

## 18. The Git routine for every implementation issue

The example below assumes actual GitHub issue **#42** exists. Replace that number with the real issue number; V-13 does not automatically mean #13. Run commands from the repository root. All paths are examples to adapt to the files actually changed.

**PM:** move the issue to Ready only when its outcome, acceptance criteria and dependencies are clear. **Developer:** move it to In progress, inspect local changes, then start from current main:

```bash
git status
git switch main
git pull --ff-only origin main
git switch -c feat/42-booking-request
```

If `git status` shows unfinished changes, commit them on their proper branch or deliberately stash them before switching. If fast-forward pull fails, inspect the divergent history; do not reset away local work.

Implement a small logical unit and review exactly what will be committed:

```bash
git diff
git add apps/api/src/modules/bookings apps/web/src packages/contracts
git diff --cached
git commit -m "feat(bookings): add dated quote requests"
git push -u origin feat/42-booking-request
```

Stage only actual relevant paths; include tests/docs when changed. Use `git add -p` to select individual hunks where helpful. Never stage secrets. `commit` records local history; `push` publishes the branch to GitHub. Neither merges it into main.

**Developer:** GitHub → Pull requests → New pull request → base `main`, compare your branch. Open a draft early if unfinished. Record why, scope, verification and rollout considerations. Use `Closes #42` only when the PR fully resolves #42. Mark ready and move the issue to In review once implementation is ready.

**Reviewer:** read Files changed independently of your coding session. Check scope, permissions, invalid inputs, error states, database/index changes and documentation. Record self-review findings as comments. **QA:** execute the issue's acceptance checks and record actual evidence.

For review fixes, stay on the same branch:

```bash
git add apps/api/src/modules/bookings
git commit -m "fix(bookings): reject duplicate active requests"
git push
```

The open PR updates automatically. Do not create a second PR for each review correction. Wait for required CI checks to pass on the latest commit. Run the project's defined verification scripts locally as relevant:

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

Add feature-specific integration/browser commands from the repository README. Passing lint/build alone is not proof that booking or payments work.

**Maintainer:** squash merge in GitHub after checks and review are complete. **Developer:** sync and clean up:

```bash
git switch main
git pull --ff-only origin main
git branch -d feat/42-booking-request
git fetch --prune
```

GitHub can delete the remote branch after merge. A squash merge may make `git branch -d` refuse because original commits are not direct ancestors. Only after verifying the PR is merged and all work exists on main, use `git branch -D feat/42-booking-request`. Do not force-delete outstanding work.

**PM:** move to Done after acceptance and merge, then demo the result. Track environment deployment/release separately; merged does not mean deployed.

## 19. Experience frontend/backend handoffs without fake people

Use the booking-request feature as one parent issue with small linked tasks:

| Sequence | Role | Assignment | Branch/PR result |
| --- | --- | --- | --- |
| 1 | PM | Define what a customer submits and sees | Parent issue with acceptance criteria |
| 2 | TL | Define request/response, errors, permissions and date rules | `docs/ACTUAL-ISSUE-booking-contract`; merge contract |
| 3 | FE | Build request form against that contract using a clearly isolated mock | `feat/ACTUAL-ISSUE-booking-form`; merge UI increment if safe/inaccessible until enabled |
| 4 | BE | Implement validation, persistence, permission and duplicate checks | `feat/ACTUAL-ISSUE-booking-api`; merge API with tests |
| 5 | FE | Pull updated main and replace mock with API | `feat/ACTUAL-ISSUE-booking-integration`; merged working journey |
| 6 | QA | Run happy path, invalid dates, duplicate and foreign-account attempts | Evidence or linked bug issue |
| 7 | PM | Demo customer request and vendor visibility | Close parent only when all acceptance criteria pass |

In a real team, steps 3 and 4 can happen concurrently after agreeing the contract. Alone, do them sequentially to limit context switching. The default for simpler features is backend → frontend in one focused feature PR; use split PRs when each increment has a clear safe boundary. Do not leave publicly reachable broken UI between merges.

Each real teammate would clone the same repository, branch from main, push their own task branch and ask another person to review. Branch ownership is task-based. Code ownership can be documented with CODEOWNERS when actual collaborators exist; it does not create people or grant repository permissions.

## 20. Updating a branch and resolving a conflict

With a clean working tree on your feature branch:

```bash
git fetch origin
git merge origin/main
```

If there are conflicts, run `git status`, open each conflicted file, read both changes and remove the conflict markers while preserving the intended combined behavior. Stage resolved files, test, then finish:

```bash
git add path/to/resolved-file
git commit
git push
```

Replace the path with each actual resolved file. To abandon the in-progress merge and return to the pre-merge state, use `git merge --abort`. Start clean so this is predictable. Avoid force-pushing a branch others use.

**Optional practice drill:** after foundation, create two disposable practice branches from the same main commit. Change the same harmless sentence in a practice document differently on each branch. Merge the first through a PR, then merge `origin/main` into the second and resolve the conflict. Do not manufacture conflicts in payment code. Use one real account and label both issues as learning exercises.

## 21. Fixing a merged bug and releasing

For a production defect, first create a bug issue with reproduction, impact and expected behavior. Branch from current main as `fix/ACTUAL-ISSUE-description`, add a meaningful regression test, fix, open a PR, verify and merge. A hotfix still gets checks appropriate to the urgency.

To undo a faulty squash-merged code change, use a new corrective branch and `git revert ACTUAL_SQUASH_COMMIT_SHA`, then review and merge that revert through a PR. Revert adds history; it does not erase shared main. Reverting application code does not undo completed payments, messages or database changes—recover those explicitly using the operational design.

After staging verifies a specific main commit, tag that exact SHA. Replace the placeholder below; do not tag a newer untested main:

```bash
git fetch origin
git tag -a v0.1.0 TESTED_COMMIT_SHA -m "Valoura sandbox MVP"
git push origin v0.1.0
```

Create release notes describing completed features, tested mode and limitations. A Git tag does not itself deploy unless you configure a workflow to do so. Deploy the same tested build; keep compatible database migrations and an explicit rollback plan.

## 22. Issue, PR and bug templates

**Feature issue example**

```markdown
Title: V-13 — Customer can request a dated vendor quote
Role: Backend, then frontend
Milestone: M3 Booking
Priority: P0
Outcome: A verified customer submits requirements to one approved vendor.
Dependencies: V-07, V-10, V-12
Scope: Request API, form, own detail view; no checkout in this issue.
Acceptance:
- Valid input creates one requested booking and displays its reference.
- Invalid dates/city and duplicate active requests are rejected.
- Customer cannot access another customer's booking.
- Vendor sees only requests addressed to its own business.
Verification: AC-03, T-07/T-08 and relevant authorization checks.
Docs: Update API contract if implementation requires a deliberate change.
```

**Pull request example**

```markdown
## Why
Customers need to send dated requirements to a selected vendor.

## Changes
Describe the actual API/UI changes and their boundaries.

## Verification
List commands actually run, outcomes and relevant screenshots.
Record the unauthorized and duplicate-request scenarios checked.

## Rollout and risks
State required indexes/configuration and any known limitation.

## Self-review
Record what was checked and corrections made; do not claim peer approval.

Closes #ACTUAL_ISSUE_NUMBER
```

**Bug issue fields:** title; environment/commit; reproduction steps; expected result; actual result; sanitized evidence; severity; related PR; acceptance for the fix. A QA failure should return the work to In progress or create a blocking bug, not be hidden in Done.

## 23. Your working rhythm and first sessions

Use one- or two-week planning windows if useful, but measure progress by completed outcomes. A solo learning project does not need formal ceremonies every day.

At the start of a window, take the PM role and select a small dependency-ready goal. Each session: read the issue → learn only the needed concept → implement one unit → verify → commit → note the next action. At the end of the window, demo the actual journey and record one process improvement. Create new issues for scope discovered during implementation rather than quietly expanding a PR.

**Session 1:** audit existing work, establish the board and merge the documentation PR (V-01).

**Session 2:** align scaffold/environment/scripts and get web plus API health running (V-02).

**Session 3:** establish foundational CI checks (V-03), then begin the first homepage component PR (V-05). If an item needs more sessions, complete it before treating its dependent work as ready.

**Next sessions:** finish fixture UI, database/contracts, verify existing auth, then follow Phases 6–14. You are not restarting completed features; you are bringing each one through the same acceptance gate.

The original backlog estimates roughly 1–2 hours per learning session and multiple sessions per issue. Treat those as planning ranges. Payments, concurrency and recovery need additional time when tests reveal failures. Phase completion is determined by its exit condition, not a deadline.

## 24. Source notes

Project decisions and V-/T-/AC- identifiers come from the previously prepared `Valoura-MVP-Complete-Guide.md`, especially documents 01, 04, 07 and 08. This workflow reorganizes those decisions; it does not replace their detailed API/state contracts.

Official references checked for this workflow on 11 September 2026:

- [S1 — GitHub Flow](https://docs.github.com/get-started/quickstart/github-flow)
- [S2 — Approving a pull request with required reviews](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)
- [S3 — Managing branch protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)
- [S4 — Next.js installation](https://nextjs.org/docs/app/getting-started/installation)
- [S5 — Razorpay Standard Checkout integration](https://razorpay.com/docs/payments/payment-gateway/web-integration/standard/integration-steps/)
- [S6 — Razorpay webhook validation and testing](https://razorpay.com/docs/webhooks/validate-test/)

Repository protection availability depends on plan/visibility. Provider live capabilities depend on account enablement. Check those in the actual account during their respective setup phases.
