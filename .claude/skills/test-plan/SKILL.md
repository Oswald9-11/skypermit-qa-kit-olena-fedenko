---
name: test-plan
description: Generate a concise, one-page QA test plan in Markdown from a requirements document, a directory of documentation, or an entire application's docs. Use this skill whenever the user asks to create, generate, draft or write a test plan (or "QA plan", "testing strategy for X") for a document, feature, module, directory or whole application, or invokes /test-plan, even if they don't say "one page" or name the output format. Works for any project.
---

# Test Plan

Produce a short, reviewable test plan that a QA lead could read in two minutes. The plan is only as trustworthy as its sources, so every statement must trace back to the documentation, or be labelled as an assumption or open question. A plan with invented requirements is worse than no plan, because people will test against it.

## 1. Pin down the source

The user's request names one of three scopes. Identify which, and read accordingly:

- **One document**: read it fully.
- **A directory**: list it, then read every document that describes behaviour (requirements, data model, state/flow, API, UI, business rules, open questions, change logs). Skip generated files and binaries.
- **The whole application**: start from the entry points (project-level instruction files, README, a docs index) and follow them. Read docs in the order the project recommends. If the docs are large, read them in full before summarising. Skimming is how contradictions get missed.

If the user names a single document, plan from that document, but check whether it points to related documents (rules, open-question trackers) it depends on. List those documents as *not reviewed* in the Sources line, so the reader knows what is unverified.

Track how thoroughly you read each source, and report it honestly in the plan's Sources line: **full** (read end to end), **partial** (only some sections read, or skimmed; say which), or **not reviewed** (referenced but not opened, or unavailable). Never imply complete documentation coverage when any file was skimmed or skipped.

If the target is ambiguous or missing, ask one short question. Otherwise proceed. Read-only: never edit the documents being planned against.

When a project keeps a "status of open questions" or "clarifications/corrections" document, treat it as the authority over the documents it corrects, and note any conflicts rather than silently choosing.

## 2. Extract before you write

Before drafting, collect from the sources, with a reference for each:

- Functional behaviour and business rules, including thresholds, limits, and ordering of checks
- Entities, states and legal/illegal transitions
- Interfaces: APIs, error contracts, roles and permissions, UI contracts
- Stated dependencies, environments, data and tooling
- Anything ambiguous, contradictory, undefined or marked open

Risk comes from this material: complexity, money or safety impact, many rules interacting, undefined behaviour, and anything marked open or corrected.

## 3. Write the plan

Use exactly these sections, in this order, as Markdown. "One page" is a hard limit: aim for about 500 words and never exceed 650, counting table text. A plan that overflows is not read, so cut rather than overflow. Budget it like this:

- Risks table: at most 6 rows (8 for a whole application). Merge related risks.
- Coverage areas: at most 8 bullets, one line each. Group rules instead of listing every one, and point to the source for detail.
- Open questions: the 6 most test-relevant, one line each (reference + impact), Business before Test design. Rank by how much they block or change expected results: a contradiction that makes a test's expected result undefined outranks a wording nit. If more exist, always end with "N further minor gaps in <source>" instead of listing them, so the reader knows the list is truncated.
- Every other section: 2-4 short bullets or lines.

Scope changes what you group, not how long the plan is. A whole-application plan covers more ground per line, so it stays at one page.

Priority rubric: **High** = failure breaks a core rule, safety/money/security, or the area is marked open, corrected or contradictory; **Med** = important rule with limited blast radius; **Low** = cosmetic or already confirmed stable. Say that priorities are derived from these criteria.

```markdown
# Test Plan: <name of document / feature / application>
*Sources: <file> (full), <file> (partial: sections), <file> (not reviewed) · Date: <today>*

## 1. Scope and objectives
2-4 sentences: what is being tested and what the plan should achieve.

## 2. In scope / Out of scope
Two short bullet lists. Out-of-scope items need a reason or "not covered by sources".

## 3. Key risks and priorities
Table: Risk | Why it matters | Priority (High/Med/Low) | Source

## 4. Testing strategy
Manual vs automated: what goes where and why, one or two lines each.

## 5. Test levels and types
Short list of levels (unit, integration, API, UI, E2E, ...) and types (functional, negative, boundary, state-transition, security, regression, ...), limited to those the sources justify.

## 6. Main coverage areas
Bullet per area, with the rules/states/endpoints it must cover and a source reference.

## 7. Environment and dependencies
Systems, services, test data, accounts, tools that the sources mention or imply (label implied ones).

## 8. Entry and exit criteria
Concise bullets. Use thresholds from the sources if given; otherwise propose sensible ones and mark them *(proposed)*.

## 9. Assumptions and open questions
Two lists. Assumptions are things you relied on that the sources don't state. Open questions are ambiguities, contradictions and gaps found in the sources, each with a reference and the testing impact. Tag each open question **[Business]** (the requirement itself is undefined, ambiguous or contradictory, so the owner of the requirements must decide) or **[Test design]** (the requirement is clear but how to test it is not: tooling, clock control, observability, test data). List Business questions first.
```

## Rules that keep the plan honest

- **Cite sources.** Use `file.md §section` (or a heading) for rules, risks and coverage areas. Plain, uncited generalities belong under Assumptions.
- **No invented requirements or expected results.** Don't state a threshold, status code, role or behaviour unless a source does. If the sources are silent, say so in Open questions.
- **Distinguish confirmed from assumed.** Mark proposals (criteria, tooling, priorities beyond what the docs imply) as *(proposed)* or *(assumption)*.
- **Verify before asking.** Before listing something as an open question, search the sources for an explicit definition, including exact comparison operators (`>=`, `<=`, "at least", "no later than", "inclusive") and any worked examples or tables. If the source defines it, it is a requirement, not a question: cite it and plan a boundary test. Only list what the sources leave genuinely undefined or contradictory.
- **Keep business and test questions apart.** Unresolved requirements belong to the requirements owner. Questions about test implementation belong to the QA team. Don't present a test-implementation question as a gap in the requirements.
- **Surface conflicts.** If two sources disagree or a doc is marked unresolved, list it in Open questions and flag affected coverage as provisional or blocked.
- **Stay project-agnostic in method.** Take every project-specific fact from the sources, never from memory or from prior projects.
- **Be concise.** Prefer tables and short bullets. Cut generic QA boilerplate that isn't tied to this source.

## Output

Show the plan in the conversation. Save it to a file only when the user asks, and in the location they name. Never save into the project being documented.
