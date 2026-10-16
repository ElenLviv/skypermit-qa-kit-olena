# Type-specific guidance

Use the section for the requested test type to shape Scope, Approach and
Criteria. The template and one-page budget stay the same. If the requested
type isn't listed, apply the same thinking: what part of the test basis does
it exercise, which techniques fit, and what does "done" mean for it.

## API testing
- **Basis:** `03-api-contract.md` (primary), `02` (legal transitions → `409`),
  `06` (gate rules → `422` body), `public/openapi.yaml` for clarification only.
- **Scope ideas:** every endpoint × auth mode (operator session, `X-Reviewer-Id`,
  none); status codes and error bodies; `422` shape (`failing_gate`,
  `rule_reference`); `validate` must not change state; idempotency/illegal
  transitions; ownership (operator A can't read/modify operator B's data).
- **Techniques:** EP/BVA on request fields, state-transition testing on
  lifecycle endpoints, decision tables on gate combinations, negative testing
  on auth headers.
- **Exit:** every endpoint and documented status code covered by ≥1 test case.

## Regression testing
- **Basis:** what changed (inspect `git log` on `skypermit/docs` and code,
  read-only; if the change isn't there, ask — see SKILL.md step 2b) + areas
  with high business impact.
- **Scope ideas:** changed areas and their neighbours (impact analysis), plus
  a stable core smoke set (register/login, create → validate → submit →
  approve, reject, cancel).
- **Approach:** risk-based selection, not "retest everything"; state which
  existing test cases are reused vs. new; confirmation testing of fixed
  defects.
- **Exit:** selected regression suite executed, no new Critical/High defects
  in unchanged areas.

## UI / frontend testing
- **Basis:** `04-frontend-spec.md`, plus `03` for the behaviour behind screens.
- **Scope ideas:** operator portal flows, reviewer queue, role switcher,
  "Check before submitting" shows per-gate results without changing state,
  pickers are deliberately *not* pre-filtered, rejection reason/rule reference
  display, `data-testid` presence and stability.
- **Out of scope by default:** visual design polish (`04` says styling is
  intentionally minimal).
- **Techniques:** use-case/scenario testing, exploratory sessions with
  charters, checklist for `data-testid` contract.

## Security / authorisation testing
- **Basis:** `03` "Authentication & authorisation", `04` (roles), `05` item on
  `SUSPENDED` operators, data model ownership rules in `01`.
- **Scope ideas:** registration/login/logout, session handling, operator data
  isolation, reviewer-only actions, suspended-operator write blocking
  (behaviour still open — risk), input validation.
- **Out of scope unless asked:** penetration testing, infrastructure security.
- **Techniques:** role × action matrix, negative/abuse cases, BVA on inputs.

## Acceptance / end-to-end
- **Basis:** all docs; focus on business rules `06` and lifecycle `02`.
- **Scope ideas:** realistic operator and reviewer journeys end-to-end,
  including rejection per gate and cancellation deadline.
- **Techniques:** scenario testing, decision tables for gate outcomes.
