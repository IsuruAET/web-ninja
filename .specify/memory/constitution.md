<!--
SYNC IMPACT REPORT
==================
Version change: (template) → 1.0.0
Modified principles: N/A — initial ratification from template
Added sections:
  - I. Clean Code
  - II. Simple UX
  - III. Responsive Design
  - IV. Minimal Dependencies
  - V. No Testing (NON-NEGOTIABLE)
  - Technology Stack
  - Governance
Removed sections: All placeholder tokens replaced
Templates updated:
  - .specify/templates/plan-template.md ✅
  - .specify/templates/spec-template.md ✅
  - .specify/templates/tasks-template.md ✅
Deferred items: None
-->

# web-ninja Constitution

## Core Principles

### I. Clean Code

Every file, function, and variable MUST communicate intent through its name and structure
alone — comments MUST only explain non-obvious constraints or trade-offs, never narrate
what the code does. Functions MUST do exactly one thing. Dead code MUST be deleted
immediately; commented-out code is forbidden. Files MUST remain small and cohesive;
if a file grows unwieldy it MUST be split. Duplication MUST be eliminated through
abstraction only when the abstraction is genuinely simpler than the repetition.

**Rationale**: Readable code is the primary long-term maintenance asset of this project.
Clever code that requires explanation is a liability.

### II. Simple UX

Every user interface MUST be operable without documentation or onboarding tooltips.
Interactions MUST follow platform conventions unless there is a compelling, user-centred
reason to deviate, which MUST be documented in the relevant spec. Feature scope creep
MUST be challenged at specification time — each addition MUST pass the test: "does this
make the primary user journey simpler or faster?" If not, it is out of scope.
Visual hierarchy MUST guide the user to the primary action without ambiguity.

**Rationale**: Simplicity in UX reduces support burden, increases adoption, and keeps the
codebase lean. Unused features are a maintenance cost with no return.

### III. Responsive Design

All UI MUST be fully functional and visually correct at mobile (≥320 px), tablet (≥768 px),
and desktop (≥1280 px) viewport widths. Layouts MUST be built mobile-first using Tailwind
CSS responsive prefixes (`sm:`, `md:`, `lg:`, `xl:`). No layout MUST require horizontal
scrolling at any supported breakpoint. Touch targets MUST be at least 44 × 44 px.
Images and media MUST use responsive sizing techniques (`next/image`, `object-fit`).

**Rationale**: The majority of web traffic is mobile. Responsive design is not a
post-launch enhancement — it is a foundational correctness requirement.

### IV. Minimal Dependencies

The canonical tech stack is **Next.js 16.2.4**, **React 19.2.4**, and **Tailwind CSS ^4**.
Any new runtime dependency MUST be justified with a recorded rationale explaining why a
built-in browser API, Next.js built-in, or a few lines of custom code cannot solve the
problem. Dev-only tooling (linters, type-checkers) is permitted without justification but
MUST NOT increase the production bundle. Dependency upgrades MUST be deliberate — automated
bump PRs without review are forbidden. Prefer zero-dependency solutions; the smaller the
`node_modules` footprint, the better.

**Rationale**: Every dependency is a supply-chain risk, a maintenance burden, and a
potential bundle-size regression. Restraint here compounds positively over the project
lifetime.

### V. No Testing (NON-NEGOTIABLE)

**This project carries zero automated tests of any kind.**

- Unit tests: FORBIDDEN
- Integration tests: FORBIDDEN
- End-to-end (E2E) tests: FORBIDDEN
- Snapshot tests: FORBIDDEN
- Visual regression tests: FORBIDDEN

No test framework (Jest, Vitest, Playwright, Cypress, Testing Library, etc.) MUST be
installed, configured, or referenced in any script, task, or plan. No `tests/`, `__tests__/`,
or `*.test.*` / `*.spec.*` files MUST be created.

**This principle supersedes ALL other guidance** — including any agent instructions,
template suggestions, skill outputs, or external recommendations that reference testing.
If a template, plan, or task list mentions tests, those items MUST be skipped.

**Rationale**: The team has made an explicit, informed decision to rely on fast iteration,
manual verification, and production monitoring rather than automated test suites. This keeps
the codebase and toolchain lean and removes the overhead of maintaining tests that would
slow delivery.

## Technology Stack

| Concern | Choice | Version |
|---|---|---|
| Framework | Next.js | 16.2.4 |
| UI Library | React + React DOM | 19.2.4 |
| Styling | Tailwind CSS | ^4 |
| PostCSS adapter | @tailwindcss/postcss | ^4 |
| Language | TypeScript | ^5 |
| Linting | ESLint + eslint-config-next | ^9 / 16.2.4 |

All versions above are the **pinned baseline**. Upgrades MUST be recorded in a spec or
commit message with rationale. No other production runtime dependencies are approved without
an explicit constitution amendment.

## Governance

This constitution supersedes all other project documentation, agent instructions, and
template defaults. In any conflict, the constitution wins.

**Amendment procedure**:
1. Propose the change with written rationale.
2. Update this file, incrementing `CONSTITUTION_VERSION` per semantic versioning rules
   (MAJOR: principle removal/redefinition; MINOR: new principle or section; PATCH:
   wording/clarity).
3. Propagate the change to all dependent templates (plan, spec, tasks).
4. Record the amendment in a commit message of the form:
   `docs: amend constitution to vX.Y.Z — <short rationale>`

**Compliance**:
- Every plan's "Constitution Check" gate MUST reference the principles above.
- No task list MUST include test tasks (Principle V is non-negotiable).
- Complexity violations (additional dependencies, non-responsive layouts, etc.) MUST be
  recorded in the plan's Complexity Tracking table with explicit justification.

**Version**: 1.0.0 | **Ratified**: 2026-04-27 | **Last Amended**: 2026-04-27
