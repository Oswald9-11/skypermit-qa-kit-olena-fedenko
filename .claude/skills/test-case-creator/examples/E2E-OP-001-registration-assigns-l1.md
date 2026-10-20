# E2E-OP-001: Registration assigns authorisation level L1 to the new operator

| Field | Value |
|---|---|
| **ID** | E2E-OP-001 |
| **Title** | Registration assigns authorisation level L1 to the new operator |
| **Doc ref** | `../skypermit/docs/03-api-contract.md` §"Authentication & authorisation" ("`authorisation_level` is no longer self-service. Registration always creates an operator at `L1`"; `GET /auth/me` returns the logged-in operator's profile) · `../skypermit/docs/04-frontend-spec.md` §"Operator portal", Login and register pages ("registration always starts an operator at `L1`") · `../skypermit/docs/01-data-model.md` §Operator (`authorisation_level` enum `L1`, `L2`, `L3`) |
| **Role(s)** | Operator (not yet registered, logged out) |
| **Type** | Positive |
| **Status** | Assumption-based |

**Preconditions**
- The app runs at `http://localhost:3000`.
- The browser has no active session (the operator is logged out).
- No operator exists with email `qa.e2e-op-001@example.com` or national ID `QA-E2E-OP-001`. Both fields must be unique (`01-data-model.md` §Operator). To re-run, change both values, for example by adding a suffix.

**Steps**
1. Open `http://localhost:3000/register`.
2. Enter these values in the form:
   - Email (`operator-register-email-input`): `qa.e2e-op-001@example.com`
   - Password (`operator-register-password-input`): `QaTest-2026-pw`
   - Legal name (`operator-register-legal-name-input`): `QA E2E Test Operator`
   - National ID (`operator-register-national-id-input`): `QA-E2E-OP-001`
   - Phone (`operator-register-phone-input`): `+380441234567`
   - Address (`operator-register-address-line-input`): `1 Test Street`
   - City (`operator-register-city-input`): `Kyiv`
   - Postal code (`operator-register-postal-code-input`): `01001`
3. Click `operator-register-submit-button`, and wait until the page leaves `/register`.
4. In the same browser window, open a new tab and go to `http://localhost:3000/api/v1/auth/me`. This sends `GET /api/v1/auth/me` with the session cookie from step 3. Read the JSON response in the tab.

**Expected result**
The JSON response from `GET /api/v1/auth/me` contains the top-level field `"authorisation_level": "L1"`.

**Rationale**
The docs say registration always starts an operator at `L1`, the least privilege, and that raising the level is an out-of-band admin action. A higher level at registration would let a new operator pass the zone gate's operator-authorisation check without any admin decision, which is a privilege-escalation risk.

**Assumptions / open questions**

*Documented requirements (cited in Doc ref):*
- Registration always creates the operator at `L1`; `authorisation_level` is not a registration field (`03-api-contract.md`, `04-frontend-spec.md`).
- `GET /auth/me` returns the logged-in operator's profile (`03-api-contract.md`).
- `authorisation_level` is one of `L1`, `L2`, `L3` (`01-data-model.md`).
- `email` and `national_id` are unique (`01-data-model.md`).

*Derived from the implementation, not documented (this is why the status is Assumption-based):*
- The `GET /auth/me` body is the operator record itself, so `authorisation_level` is a top-level field (`app/api/v1/auth/me/route.ts`). The docs don't specify the body.
- The register form's `data-testid` values come from `app/register/page.tsx`. `04-frontend-spec.md` doesn't list them.
- The password must be at least 8 characters (`app/api/v1/operators/route.ts` and the form's `minLength`). The docs state no password rule.

*Assumed by the test author:*
- The base URL `http://localhost:3000` (the dev-server address in the application's `CLAUDE.md`).
- The test-data values are valid. The docs give no format rules for these fields.
- The success status code of registration is not documented, so this case doesn't assert one.
