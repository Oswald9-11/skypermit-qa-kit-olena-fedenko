# Test Plan: SkyPermit (whole application)
*Sources: skypermit/CLAUDE.md, docs/01-08 (all read; 08 skimmed, derived from 01/02) · Date: 2026-10-20*

## 1. Scope and objectives
Verify that registration, registry, flight-request lifecycle, the four gates, reviewer decisions and both portals behave as documented, and flag areas the docs leave undefined.

## 2. In scope / Out of scope
- In: `/api/v1` endpoints 1-16 plus auth and list extensions; state machine; gates; derived values; operator and reviewer portals; migrations/seed zone data.
- Out: `CR-1` and `OH-1` v1.4 values (not in force, 06); admin level changes (out-of-band, 03); visual design (04).

## 3. Key risks and priorities
Priorities derived from the rubric (High = core rule/security or open/corrected/contradictory; Med; Low).

| Risk | Why it matters | Priority | Source |
|---|---|---|---|
| Gate order/short-circuit wrong | `failing_gate` must be the first failure in PILOT→AIRCRAFT→ZONE→TIME_WINDOW | High | 03 Gate pipeline |
| Weight-band check in wrong gate | Corrected from ZONE to AIRCRAFT; shared `ZC-2026.1 §2` citation with altitude | High | 07 §1, 06 §1.2 |
| Illegal transitions not returning 409; cancel deadline | Any unlisted pair must be 409; APPROVED cancel only until 2h before start | High | 02 |
| Authz/session leaks | Server-derived `operator_id`; 401 vs 403; reviewer header scope | High | 03 Auth |
| Time rules: UTC/curfew, overlap | Minute-level curfew across midnight, `A-7` override, touching windows, only APPROVED/UNDER_REVIEW counted | High | 06 §1.4, 01 |
| SUSPENDED operator and zone-active citation | Both open; no defined behaviour | High | 05 §1-2 |
| Derived values stored or stale | Currency and band must be computed at read time | Med | 01 Derived |
| UI contract drift | `data-testid` stability; pickers must not pre-filter | Med | 04 |

## 4. Testing strategy
- Automate (Vitest, per `CLAUDE.md` `tests/`): gate logic, boundaries, state transitions, route status codes, 422 body.
- Manual/exploratory: portals, open-question areas. UI E2E on `data-testid` *(proposed)*.

## 5. Test levels and types
- Unit (gates), integration/API (routes, DB constraints, audit trigger), UI/E2E (portals).
- Functional, negative, boundary, state-transition, security (401/403), regression (selectors, corrected rules).

## 6. Main coverage areas
- PILOT: rating vs class; 18-month currency vs `window_start`, no grace (06 §1.1, §2).
- AIRCRAFT: bands C0-C3 boundaries 900/3,500/12,000/30,000 g; >30,000 g refused with 422 at registration (06 §3).
- ZONE: altitude ≤ ceiling; auth level L1<L2<L3; inactive zone; six-zone catalogue (06 §1.3, §4).
- TIME_WINDOW: ≥6h lead, ≤60 days, 15-240 min inclusive, curfew, overlap (06 §1.4).
- `rule_reference` per condition, verbatim (06 §6); 422 body shape (03).
- State machine and `validate` has no side effects; submit never rests in SUBMITTED (02, 03).
- Reviewer: missing `X-Reviewer-Id` gives 400; reject note required; ReviewAction, append-only AuditLog (01, 03).
- Auth, ownership, list scoping, 400/404 contracts; `flight_purpose` ≤2,000 chars (03, 01).
- Portals: login/register (always L1), pickers, queue, selectors (04).

## 7. Environment and dependencies
- Next.js app, Supabase project (Auth, Postgres), `.env.local` keys, migrations 0001-0004 and `seed.sql` (CLAUDE.md, SETUP.md).
- Test data: L1-L3 users, all ratings, edge masses, lapsed certifications, inactive zone; clock control *(implied)*; `tests/fixtures.ts`.

## 8. Entry and exit criteria
- Entry: build passes, migrations and seed applied, test accounts exist.
- Exit: all High areas executed, no open High/critical defects, `npm run lint` and `npm test` pass *(proposed)*.
- Open-question areas reported as provisional.

## 9. Assumptions and open questions
Assumptions:
- `SETUP.md`, `OH-1`, `TSD-4`, `ZC-2026.1`, `A-7` not read; unverified.

Open questions:
- SUSPENDED operator status code/body undefined (05 §1); tests blocked.
- Zone-active citation `ZC-2026.1 §4` unconfirmed (05 §2); assertion provisional.
- Overlap check placed in "AIRCRAFT / TIME_WINDOW gates" (02) but only TIME_WINDOW (06 §1.4); affects `failing_gate` expectation.
- Cancel after deadline: "should" return 409 (02) vs "rejected" (06 §5); status confirm.
- Lead-time and max-booking boundary inclusivity unstated (06 §1.4).
