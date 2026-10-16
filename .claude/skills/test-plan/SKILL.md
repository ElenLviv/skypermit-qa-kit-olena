---
name: test-plan
description: Generate a one-page test plan (ISTQB / ISO/IEC/IEEE 29119-3, condensed) for the SkyPermit project from its test basis in skypermit/docs. Use this whenever the user asks for a test plan, test strategy, test approach, QA plan, or "what/how should we test" for SkyPermit as a whole (release, MVP) or for one test type (API, regression, UI/frontend, security/authorisation, integration, acceptance, exploratory, etc.) — even if they don't say "test plan" explicitly, e.g. "plan the testing for the next release" or "how should we approach API testing". Not for writing individual test cases or automated tests.
---

# One-page test plan for SkyPermit

Produce a concise, decision-ready test plan that fits on one page, follows the
structure of ISO/IEC/IEEE 29119-3 (condensed), and is traceable to the
SkyPermit test basis. The reader is a QA lead or stakeholder who wants to see
scope, approach, risks and criteria at a glance — not a 20-page document.

## Context you must respect

- **System under test:** the `skypermit` project, a sibling folder of this qa-kit
  (the exact path is in the qa-kit `CLAUDE.md`). It is **read-only** — read
  files and run read-only git commands there, nothing else.
- **Test basis:** `skypermit/docs/` (01–08). This is the primary source. Code
  (`app/`, `lib/`, `supabase/`, `public/openapi.yaml`) may be read only to
  clarify behaviour; if code and docs disagree, that is a risk/potential
  defect to report, not a fact to plan around.
- **Approval:** the qa-kit rule is to ask before creating or changing any file.
  Show the plan first, then write it after the user approves.
- **No invention:** team members, dates, environments, tools, budgets are not in
  the docs. Use `[TBD: …]` placeholders and collect them as questions — a plan
  with made-up names or dates is worse than one with honest gaps.

## Workflow

### 1. Pin down scope and type
From the request, decide:
- **Scope:** whole product / release (default), or a test type such as API,
  regression, UI, security/authorisation.
- **Release/iteration name** if given; otherwise `[TBD: release]`.

Only ask a clarifying question if the request is genuinely ambiguous (e.g.
"test plan for the changes" with no indication of which changes). Otherwise
proceed and state your interpretation in the plan's Context line.

For a specific test type, read `references/test-types.md` for type-specific
scope, techniques and criteria.

### 2. Read the test basis
The docs are short (~1000 lines total) — read all eight files for a
whole-product plan, and at least the relevant ones plus `05-open-questions.md`
and `07-clarifications.md` for a type-specific plan. Things to extract:

- **Features / test items** — operator registry (pilots, aircraft), auth
  (register/login), flight request lifecycle, the four gates (PILOT, AIRCRAFT,
  ZONE, TIME_WINDOW), validate vs submit, reviewer queue, cancellation,
  audit log, reference data (zones), UI `data-testid` contract — but derive
  this from the docs as they are now, they may have changed.
- **Product risks** — complex rules (gate thresholds, time windows, curfew,
  overlap), state transitions and `409`s, authorisation boundaries
  (operator owns only its data, reviewer header), time handling/time zones,
  error contract (`422` body), and anything marked open or ambiguous.
- **Open questions** — every item still *Open* in `05-open-questions.md` is a
  project risk and must appear in the Risks section. Also flag contradictions
  you notice between documents.

Record the test-basis version so the plan can be re-checked when docs change:
```bash
git -C <skypermit-path> log -1 --format="%h %ad" --date=short -- docs
```

### 2a. Cross-check the documents against each other
The docs were written at different times and corrected piecemeal, so they
contradict each other in places. Finding those contradictions before test
design is one of the most valuable things a test plan can do — each one is a
risk (and later a potential defect or a question for the requirements owner).
Compare the documents deliberately, especially:

- **Error codes** — does the same situation get different status codes in
  different docs (e.g. `409` in the API contract vs `422` gate failure in the
  business rules; `400` vs `401` for a missing header)?
- **Response shapes** — does one doc describe a response differently from
  another (e.g. "stops at first failure" vs "shows all gate results")?
- **Referenced but undocumented** — actions, endpoints or fields mentioned in
  one doc (lifecycle, UI) with no definition in the API contract or data model.
- **Unreachable or untestable** — legal transitions from states that can never
  be observed, rules with no data to exercise them (e.g. no inactive zone in
  the seed).
- **Status drift** — something `05-open-questions.md` calls *Open* that another
  doc already treats as settled, or vice versa.

Put the significant ones in Risks (with both references, e.g. `[03 Errors]` vs
`[06 §1.4]`) and turn each into an open question.

### 2b. Regression and change-based plans
A regression plan depends on knowing what changed. Check the git history of
`skypermit` (read-only: `git log`, `git diff`) for the change the user names.
If the change isn't visible there or in the docs, don't guess at it: plan the
regression approach generically (impact analysis, core smoke set, confirmation
tests), mark the change set and affected areas as `[TBD: change list]`, make
"change set unknown" the top risk, and ask for the PR/ticket/change list.

### 3. Write the plan
Use the template in `assets/test-plan-template.md` exactly in its section
order. Guidance for each section:

- **Context & objectives** — one or two sentences: what is tested, why, and
  which test basis version. Objectives are outcomes (e.g. "confirm all four
  gates reject per business rules with correct `rule_reference`"), not
  activities.
- **Scope** — in-scope features and out-of-scope items as short bullets, each
  with a source reference like `[06 §1.2]`. Out-of-scope must be explicit
  (e.g. performance, visual design — `04` says styling is intentionally
  minimal) so nobody assumes it is covered.
- **Approach** — test levels and types, test design techniques mapped to the
  features they suit (state transition → lifecycle; BVA/EP → thresholds and
  time windows; decision tables → gate combinations; role-based → auth),
  test data needs, and the risk-based priority order. Mention that dev-owned
  automated unit/route tests exist in `skypermit/tests/` and how manual
  testing complements them, but don't plan automation work.
- **Risks** — 4–7 rows max, highest first: risk, likelihood/impact (H/M/L),
  mitigation. Include project risks (open questions, missing environment)
  as well as product risks.
- **Entry / exit criteria** — measurable (e.g. "100% of in-scope requirements
  have ≥1 test case", "no open Critical/High defects", "all `05` open items
  answered or accepted as known risk").
- **Environment, roles & schedule** — `[TBD]` unless the user told you; list
  what is needed (e.g. Supabase project with seed data, reviewer IDs, clock
  control for time-window/cancellation-deadline tests).
- **Deliverables** — test conditions, test cases, defect reports, test summary
  report, traceability matrix.

### 4. Keep it to one page
One page ≈ 450–600 words of body text (count everything above the Open
questions rule; tables included). Use tables and terse bullets; cut
explanations. Detail belongs in later artifacts (test cases, traceability
matrix), not here. If something important doesn't fit, it's a sign that the
plan should reference it rather than contain it.

The **Open questions for the user** list goes *below* the plan, after a
horizontal rule — it is not part of the one-page budget.

### 5. Present and save
1. Show the full plan in chat, followed by the open questions.
2. Propose the file path `test-plans/TP-<scope-slug>.md` inside the qa-kit
   (e.g. `TP-release-mvp.md`, `TP-api.md`, `TP-regression.md`) and ask for
   approval to write it. If a file with that name exists, propose a new
   version rather than overwriting.
3. After approval, write the file. Don't commit unless asked.

## Quality bar (check before presenting)
- Every in-scope feature and every risk has a `[doc §]` reference.
- All *Open* items from `05-open-questions.md` appear as risks.
- Significant cross-document contradictions appear as risks and open questions.
- No invented people, dates, environments or tools — only `[TBD: …]`.
- Fits one page; Open questions are outside the page.
- Exit criteria are measurable.
