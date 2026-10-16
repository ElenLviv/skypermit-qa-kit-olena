# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## Context

`skypermit-qa-kit-olena` is the QA kit for the **SkyPermit** project — a Next.js + Supabase application for drone flight permit requests (operator portal, reviewer queue, gate-based request evaluation).

- **System under test (SUT):** `C:\Users\olena.kliushnyk\Kliushnyk-work\AI Personal\Claude\workspace\skypermit` — **READ-ONLY**
- **Test basis:** `C:\Users\olena.kliushnyk\Kliushnyk-work\AI Personal\Claude\workspace\skypermit\docs`
- **This repository** holds the QA artifacts produced from that test basis: test plans, test conditions, test cases, and related test documentation.

Claude is used here for **test plan generation and test case generation** (manual test design). Claude is **not** used for writing or maintaining automated tests in this kit.

## Goal

Organize and support the SkyPermit test process in line with the **ISTQB test process**, producing traceable, reviewable test artifacts derived from the test basis:

1. **Test planning** — define test objectives, scope (in/out), test levels and types, approach, entry/exit criteria, risks, resources, and deliverables.
2. **Test monitoring and control** — track progress against the plan, measure coverage of the test basis, report status, and propose corrective actions when deviations occur.
3. **Test analysis** — analyse the test basis to identify testable features and derive test conditions ("what to test"); detect defects in the requirements themselves (ambiguities, contradictions, gaps, untestable statements) and cross-check against `05-open-questions.md` and `07-clarifications.md`.
4. **Test design** — turn test conditions into test cases ("how to test") using appropriate techniques (equivalence partitioning, boundary value analysis, decision tables, state transition testing, use-case/scenario testing, experience-based techniques); identify required test data and environment.
5. **Test implementation** — organize test cases into test suites and execution order, prepare test data and preconditions, and confirm readiness for execution.
6. **Test completion** — summarize results and coverage, record lessons learned, document open issues and residual risks, and archive the testware.

Every test artifact must stay **traceable** to its source in the test basis (document + section), so coverage can be measured and gaps identified.

## Test basis — SkyPermit `docs/`

| Document | Content |
|---|---|
| `01-data-model.md` | Entities (Operator, Aircraft, Pilot, Zone, Flight Request, …) and their attributes |
| `02-state-machine.md` | Flight request lifecycle — states and transitions |
| `03-api-contract.md` | REST API contract, authentication & authorisation, registry endpoints |
| `04-frontend-spec.md` | Frontend scope — operator portal, reviewer queue |
| `05-open-questions.md` | Open questions — canonical status |
| `06-business-rules.md` | Business rules — evaluation order, the four gates' pass/fail logic |
| `07-clarifications.md` | Answers from the business rules owner |
| `08-uml-diagrams.md` | ER diagram, flight request state diagram, other UML |

`07-clarifications.md` resolves items from `05-open-questions.md`; when documents conflict, flag the conflict to the user rather than silently choosing one.

## SkyPermit directory structure (read-only reference)

```
skypermit/
├── .claude/settings.json
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── app/                          # Next.js App Router
│   ├── api/
│   │   ├── items/
│   │   └── v1/                   # REST API: aircraft, auth, flight-requests, operators, pilots, zones
│   ├── api-docs/page.tsx
│   ├── swagger/page.tsx
│   ├── login/page.tsx
│   ├── register/page.tsx
│   ├── operator/                 # Operator portal
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── aircraft/page.tsx
│   │   ├── pilots/page.tsx
│   │   └── requests/             # list, new, [id]
│   ├── reviewer/                 # Reviewer queue
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── requests/[id]/
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── components/                   # React components
│   ├── ApiDocs.tsx, AppFooter.tsx, AppHeader.tsx
│   ├── FlightRequestPanel.tsx, GateResultsPanel.tsx
│   ├── OperatorGate.tsx, RequireReviewer.tsx
│   ├── identity.tsx, session.tsx
│   └── ui/                       # Button, Field, GateCard, StateBadge, Toast, icons, primitives
├── docs/                         # TEST BASIS (see table above)
│   ├── 01-data-model.md
│   ├── 02-state-machine.md
│   ├── 03-api-contract.md
│   ├── 04-frontend-spec.md
│   ├── 05-open-questions.md
│   ├── 06-business-rules.md
│   ├── 07-clarifications.md
│   └── 08-uml-diagrams.md
├── lib/                          # Domain logic
│   ├── gates/                    # aircraft-gate, pilot-gate, time-window-gate, zone-gate, pipeline, types
│   ├── client/api.ts
│   ├── auth.ts, curfew.ts, derived.ts, flight-requests.ts, gate-context.ts
│   ├── http.ts, state-machine.ts, types.ts, validation.ts
│   └── supabase.ts, supabase-admin.ts, supabase-server.ts
├── public/                       # static assets, openapi.yaml
├── supabase/
│   ├── migrations/               # 0001_init … 0004_operators_id_is_auth_user
│   ├── seed.sql
│   └── SETUP.md
├── tests/                        # Dev-owned automated tests (vitest) — reference only
│   ├── business-rules/
│   ├── lib/ (gates/, curfew, derived, state-machine, validation)
│   ├── routes/ (aircraft, submit)
│   └── fixtures.ts
├── package.json, tsconfig.json, next.config.ts, vitest.config.mts
└── eslint.config.mjs, postcss.config.mjs
```

Source code under `app/`, `lib/`, `components/`, `supabase/` and `public/openapi.yaml` may be read to clarify behaviour, but the **requirements in `docs/` are the primary test basis**. Where implementation and docs disagree, report it as a potential defect — do not treat the code as the oracle.

## Constraints

- **The SkyPermit project is READ-ONLY.** Never create, modify, move, or delete any file under `workspace\skypermit` — no edits, no generated files, no git operations, no `npm install`/build/test runs that write to that folder.
- **Ask for user approval before any change in this qa-kit repository.** Before creating, editing, renaming, or deleting any file here, describe the intended change (files and summary) and wait for explicit approval.
- No automated test code is produced in this kit; deliverables are test documentation only.
