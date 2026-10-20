# Test Plan: Request Lifecycle (FlightRequest state machine)
*Sources: skypermit/docs/02-state-machine.md · Date: 2026-10-20*

## 1. Scope and objectives
Verify that a `FlightRequest` moves only through the legal transitions of the seven-state lifecycle, that every other `(from, action)` pair returns `409`, and that the cancellation-deadline and schedule-overlap rules behave as described. Priorities below are derived from the High/Med/Low criteria (core rule or money/safety, or open/contradictory = High).

## 2. In scope / Out of scope
- In: state table, legal/illegal transitions, terminal states, reviewer-only and maintenance-only actions, cancellation deadline, overlap visibility by state, single write path and audit trigger (02-state-machine.md, all sections).
- Out: internal logic of the four gates, and business-rule details beyond the cancellation deadline (not covered by this source; `06-business-rules.md` not read).

## 3. Key risks and priorities
| Risk | Why it matters | Priority | Source |
|---|---|---|---|
| Illegal transition accepted (e.g. submit from `UNDER_REVIEW`) | Breaks the core "anything not listed is 409" rule | High | §Request Lifecycle |
| Cancel of `APPROVED` near `window_start` | Deadline of 2h; wording is "should reject" | High | §Rules elsewhere |
| Terminal states (`REJECTED`, `CANCELLED`, `COMPLETED`) leave their state | Irreversible outcomes | High | §States |
| Overlap check counts wrong states | Double-booked aircraft; validate-pass/submit-fail is expected | High | §Rules elsewhere |
| Non-reviewer approves/rejects; reject without `note` | Authorisation and audit gap | High | transition table |
| State written outside the single path / no audit row | Missing audit trail | Med | §Implementation note |

## 4. Testing strategy
- Automated: table-driven transition matrix (every state x every action), deadline boundary, overlap by state. Cheap, exhaustive, regression-prone.
- Manual/exploratory: concurrency (approve vs cancel), the diagram-versus-table reading, audit trigger on real DB.

## 5. Test levels and types
- Unit (transition function), API/integration (endpoints, DB trigger).
- Functional, negative (illegal pairs), boundary (2h deadline), state-transition, authorisation, regression.

## 6. Main coverage areas
- All 10 legal rows of the transition table (§Request Lifecycle).
- Full illegal matrix returning `409`, incl. terminal states and re-submit (§Request Lifecycle).
- Submit outcome: never observable at rest in `SUBMITTED`; `failing_gate`/`rejection_reason` set on reject (§States).
- Approve/reject: reviewer-only via `X-Reviewer-Id`, `ReviewAction` row, `note` required on reject.
- Complete: only via maintenance endpoint, never from elapsed time.
- Cancel from `DRAFT`/`SUBMITTED`/`UNDER_REVIEW`/`APPROVED`, with 2h cut-off.
- Overlap: only `APPROVED` and `UNDER_REVIEW` count (§Rules elsewhere).
- Audit: every state change logged by the trigger (§Implementation note).

## 7. Environment and dependencies
- Database with `supabase/migrations/0001_init.sql` applied (named in source).
- Reviewer and operator identities; maintenance endpoint access *(implied)*.
- Controllable clock or `window_start` fixtures for deadline tests *(implied)*.

## 8. Entry and exit criteria
- Entry: build deployed, migration applied, test data seeded *(proposed)*.
- Exit: 100% of legal and illegal state/action pairs executed; no open High defects; boundary and overlap tests pass *(proposed)*.

## 9. Assumptions and open questions
**Assumptions**
- `409` is the only required signal; response body shape is untested.
- Referenced `06-business-rules.md` §5, `TSD-4 §6`, `0001_init.sql` and gate docs were not read; unverified.

**Open questions**
1. Cancel at exactly 2h before `window_start`: allowed or rejected? Source says "up to" and "should reject". Boundary expectation unknown.
2. `SUBMITTED` cancel is listed legal yet submit is synchronous and never at rest; how is it testable?
3. Diagram reject arrows appear to include `APPROVED`; table allows only `UNDER_REVIEW`. Table treated as authority, provisionally.
4. Unauthorised approve/reject: status code not stated (401/403/409?).
5. Concurrent actions (cancel vs approve): expected winner undefined.
6. Does a request overlap-check against itself or its own earlier state? Not stated.
