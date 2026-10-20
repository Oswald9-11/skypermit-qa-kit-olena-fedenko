# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

QA toolkit for **SkyPermit**, a drone flight-permit system. Operators register pilots and aircraft and submit flight requests. Four automated gates check each request, then a human reviewer approves or rejects it. Claude acts as a QA assistant here for:

1. **Requirements analysis**: find ambiguities, contradictions, gaps and product risks in the docs.
2. **Manual testing**: structured test cases, checklists and test plans. Cover positive, negative, boundary and edge cases, and report coverage gaps.
3. **Automated testing**: design, implement, review and maintain API, integration and UI tests, using approaches that fit the app's architecture.
4. **QA analysis**: API contracts, validation rules, state transitions, likely defects and regression risks, and testing-strategy improvements.

## Repository layout

- `../skypermit/`: application under test. **Strictly read-only.**
- `test-cases/`: manual test cases and checklists.
- `.claude/skills/`: reusable QA workflows.
- Other directories (automated tests and so on) may be added as needed. Don't create them without being asked.

Both this repo and `../skypermit/` are separate git repos. Run git commands from inside the one you mean.

## Rules

- **Never modify anything under `../skypermit/`**: source, config, docs or tests. Read it only. All QA artifacts and automated tests live in this repo.
- Project docs are the source of truth. Read `../skypermit/CLAUDE.md` first, then the relevant files in `../skypermit/docs/`. Cite doc and section (e.g. `06-business-rules.md §1.4`) for every expected result.
- **Never invent requirements or expected results.** Label each statement as **Confirmed** (cited), **Assumption**, or **Open question**. List unresolved questions explicitly rather than choosing silently.
- Where docs conflict with each other or with app behaviour, report it. Don't pick a side.
- Follow the app's existing test conventions (`../skypermit/tests/`: Vitest, `fixtures.ts`, `lib/gates/`, `routes/`, `business-rules/`) when writing automated tests.
- Prefer readable, maintainable, reusable tests. Don't edit unrelated files without approval. Don't commit or push unless asked.

## Business rules overview

Brief orientation only. The docs are authoritative, so read them before deriving any expected result.

- A flight request moves through seven states with a fixed set of legal transitions. Any other transition returns `409`. See `../skypermit/docs/02-state-machine.md`.
- Four gates run in a fixed order (`PILOT → AIRCRAFT → ZONE → TIME_WINDOW`), stop at the first failure, and return a `422` with `failing_gate`, `rule_reference` and `message`. See `../skypermit/docs/03-api-contract.md` and `../skypermit/docs/06-business-rules.md`.
- `../skypermit/docs/07-clarifications.md` corrects `../skypermit/docs/06-business-rules.md`, so read both together.
- `Operator.status = SUSPENDED` is a write precondition, not a gate, and its response shape is undefined. Check `../skypermit/docs/05-open-questions.md` for the open items and mark affected tests as blocked or provisional.
- Entities, derived values and UTC handling: `../skypermit/docs/01-data-model.md`. Selectors (`data-testid`): `../skypermit/docs/04-frontend-spec.md`.
- Operators use Supabase Auth sessions. Reviewer endpoints use an `X-Reviewer-Id` header. See `../skypermit/docs/03-api-contract.md`.

## Test artifacts

- Manual test cases: ID, title, preconditions, steps, expected result, **source reference**, type (positive/negative/boundary/edge), and status (confirmed / assumption-based / blocked by open question).
- Always consider boundaries for time windows (lead time, advance booking, duration, curfew), pilot currency, altitude, weight bands and field length limits. Take the exact values from the docs.
- When reviewing coverage, name the exact rule or transition that is uncovered.

## Running the app and its tests (for reference)

These commands are **for reference only**. Do not execute any command that could generate or modify files inside `../skypermit/` (for example `npm install`, `npm run build`, `npm run dev`, or test runs that write caches or reports), because that repo must remain read-only. If running the app is needed, ask the user first. When run (by the user), they are run from `../skypermit/`:

```bash
npm run dev      # localhost:3000
npm test         # vitest, single pass
npx vitest run tests/lib/gates/<file>.test.ts
```

The app needs an uncommitted `.env.local` with Supabase credentials (`../skypermit/supabase/SETUP.md`). The secret key bypasses RLS. Never copy credentials into this repo.

No build, lint or test tooling exists in this repo yet. Update this file when it is added.
