---
name: test-case-creator
description: Create exactly ONE atomic, low-level manual test case (one condition or action, one observable expected result) from a requirement, feature or specific application behaviour, grounded in the project's documentation. Use this skill whenever the user asks to write, create, draft or generate a test case, a single test, a boundary/negative/edge-case check, or a manual test for a rule, endpoint, screen or state transition, or invokes /test-case-creator, even if they don't say "atomic" or name the format. For several tests, a checklist or a test plan, do not use it; one invocation produces one case. Works for any project.
---

# Test Case Creator

Write one atomic manual test case: it verifies one behaviour and has one observable expected result. A result may have several closely related assertions (for example a status code, an error code and the unchanged state after one rejected action) when they all describe that one behaviour. If one case would need checks for separate, independently testable requirements, those are separate cases. Splitting keeps failures easy to diagnose, because a failed case then points at one cause. This skill produces one case per invocation, so offer the other checks as follow-ups rather than bundling them.

The documentation is the only source of truth. A test case with an invented expected result is worse than none, because people will treat a correct behaviour as a defect, or the reverse.

## 1. Pin down the target

Before writing, you need three things. Ask for what's missing, in one short message, and do not guess:

1. **The behaviour to check.** If the request bundles several checks ("test the whole submit flow", "validate all fields"), ask which single check to write first, and list the others as candidates for later.
2. **The documentation source.** Use the file(s) the user names. If none are named, ask which docs to use. Never fall back on memory or prior projects.
3. **The module** the case belongs to (used for the folder and the ID). Take it from the docs' structure where clear; otherwise ask.

If the target behaviour is ambiguous after reading the docs, ask for clarification before writing. Only after the docs are read can you tell whether it is ambiguous.

## 2. Read the docs and verify the facts

Read the cited section in full, plus anything it depends on (rule tables, error contracts, state tables, clarification or open-question documents, which override the documents they correct). Then verify, against the text:

- **Exact values:** thresholds, limits, units, enum names, field names, and the comparison operator (`>=`, `<=`, "at least", "no later than", "inclusive"). A boundary case needs the exact boundary and which side passes.
- **Exact expected outcome:** status code, error body fields, state change, message text, or UI element, as written. Don't paraphrase a value you can quote.
- **Roles and permissions:** use only roles the docs define.
- **Open or contradictory points:** if the docs leave the expected result undefined or conflict, don't pick a side. The case is **Blocked** (see Status).

Before treating anything as unresolved, search the docs for an explicit definition. If it is defined, it is a requirement.

## 3. Write the test case

Before writing, consult the completed test cases in `examples/` (in this skill's directory), when any are available. Use them as a reference for format, level of detail, executable steps with concrete test data, and for separating documented requirements from implementation-derived details and test-author assumptions. The examples come from specific projects: never copy their facts, values or wording into a case for another feature; take every fact from the documentation you read for the current request. Add further examples to `examples/` only when the user approves them.

Use `templates/test-case.md` (in this skill's directory) as the primary structure: read it, then fill in every placeholder. Keep its sections, their order and their field names exactly. Replace each `<placeholder>` with real content, choose a single value where a placeholder lists options (Type, Status), and delete the template's comment. Keep the **Assumptions / open questions** section only when Status is not Confirmed; omit it when Status is Confirmed. Add nothing the template doesn't contain.

### Field rules

- **ID:** `{LEVEL}-{MODULE}-{NNN}`, e.g. `E2E-OP-001`, `API-GATE-003`.
  - LEVEL is the test level (E2E, UI, API, INT, UNIT); MODULE is a short uppercase code for the module. Derive both from the request, and state them to the user when you confirm.
  - NNN is the next free number: scan `test-cases/{module}/` for existing IDs for that LEVEL and MODULE and take the highest plus one (`001` if none). Never reuse an ID.
- **Title:** short and specific: the condition and the outcome, not "Test submit".
- **Doc ref:** exact file and section (or heading). A bare file name is not enough.
- **Role(s):** only roles the docs define. If the docs define none, write `Not specified in docs` and say so.
- **Preconditions:** the state and data needed so the case runs on its own, with concrete values (including the boundary value). The case must not depend on another case having run first.
- **Steps:** the minimum numbered actions to reach and trigger the condition. No branching, no "verify" steps that duplicate the expected result.
- **Expected result:** one outcome of one behaviour that an observer can verify without judgement; list its closely related assertions together when they all belong to that behaviour. Quote exact codes, field values, states or messages from the docs. Where the docs only say a thing happens, state only that; don't add detail.
- **Type:** Positive (valid input succeeds), Negative (invalid input is refused), Boundary (at/just either side of a defined limit), Edge case (unusual but valid situation).
- **Status:**
  - **Confirmed**: every fact in the case is stated in the docs.
  - **Assumption-based**: at least one detail (data value, step, environment) is assumed. List each.
  - **Blocked by open question**: the expected result is undefined or contradictory in the docs. Give the expected result as "Undefined in docs" with the reference, rather than a guess.

## 4. Quality gate before showing it

Check the case against this list and fix it silently:

- One behaviour, one expected result. The word "and" is not a reason to split: several closely related assertions about the same behaviour are fine. Split only when the case checks separate, independently testable requirements, i.e. one could pass while the other fails for an unrelated reason or the docs define them as separate rules. Keep only the first and offer the rest as follow-ups.
- No vague wording: "works correctly", "displays an error", "is handled properly", "as expected".
- Every value, code and name appears in the cited docs, or is marked as an assumption.
- A reader could run it cold: preconditions give state and data; steps use no hidden knowledge.
- Doc ref points to a real section you read.

## 5. Confirm, then save

Show the case in the conversation first. Ask the user to confirm before saving, and mention any other single checks you noticed as follow-ups.

On confirmation, save one case per file:

- **Path:** `test-cases/{module}/{ID}-{kebab-case-title}.md`, with `{module}` a lowercase folder name (e.g. `operator`). Create the folder if needed.
- **Example of the convention:** `test-cases/operator/E2E-OP-001-submit-rejected-when-pilot-currency-expired.md`.
- Keep the filename's kebab-case title in step with the Title field.
- Save only into the QA repository, never into the project being documented. Do not overwrite an existing file; if the ID is taken, pick the next number.

Do not save without confirmation.
