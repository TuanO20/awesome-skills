---
name: best-tdd-workflow
description: "Use this skill whenever implementing a new feature end-to-end, especially ones that touch an API contract, database entities, or multiple callers. It is a six-phase workflow that emphasizes planning, testing, and incremental development."
---

# Feature Implementation Workflow

Follow this six-phase flow when implementing any new feature. Each phase has a
**gate**: do not advance until the gate is satisfied. If a gate fails, stay in
the phase or step back — never skip ahead.

Keep changes surgical. Every changed line should trace directly to the feature.
State assumptions before coding; if intent is ambiguous, stop and ask.

---

## Phase 1 — Plan & Impact Analysis

**Goal:** Understand the change fully before touching code.

- Read the relevant existing code, specs, and data models first.
- Define the feature's success criteria in verifiable terms (inputs, outputs,
  observable behavior).
- Map the blast radius: which modules, endpoints, database entities, and
  callers are affected (use Gitnexus MCP if have).
- List assumptions and open questions. Surface tradeoffs and simpler
  alternatives. Ask if anything is unclear.
- Produce a short numbered plan, each step paired with how it will be verified.

**Gate:** Plan is written, assumptions are stated, and success criteria are testable. No coding before this is done.

---

## Phase 2 — Update OpenAPI Specs

**Goal:** Define the contract before the implementation.

- Add or update paths, request/response schemas, status codes, and error
  shapes in the OpenAPI spec.
- Keep naming, versioning, and error conventions consistent with the existing
  spec.
- Treat the spec as the source of truth that tests and implementation follow.

**Gate:** Spec is updated and valid (lints/parses cleanly), and it fully
describes the new behavior including error cases.

---

## Phase 3 — Write Tests (cover all cases)

**Goal:** Encode the contract as failing tests before implementing.

- Choose only the test types needed (unit: business logic, component: UI components, integration: system interactions, end-to-end: only for critical user journeys).
- Write normal-path tests for expected inputs and outputs.
- Write edge-case tests: empty/null, boundary values, invalid input,
  auth/permission failures, conflicts, duplicated requests, concurrent operations,
  and error responses from the spec.
- Aim to cover every documented case in the OpenAPI contract.
- Tests must fail for the right reason (feature not implemented yet), not
  because of setup errors.

**Gate:** Tests exist for normal and edge cases, run, and fail because the
feature is not yet implemented.

---

## Phase 4 — Implement (pass all tests)

**Goal:** Write the minimum code that makes the tests pass.

- Implement the simplest solution that satisfies the spec and tests.
- Take care database safety (backward compatibility, indexes, constraints), concurrency & idempotency (duplicate requests, race conditions, retries, lost updates, double execution), observability (show logging, metrics, tracing if important to fix issues), and security (auth, permissions, input validation).
- No features, abstractions, or configurability beyond what was asked.
- Match existing project style, patterns, and libraries. Don't introduce new
  ones without reason.
- Run tests and iterate until green.

**Gate:** All tests pass. The build/compile step runs clean.

---

## Phase 5 — Fix & Refactor

**Goal:** Improve quality without changing behavior.

- Run linters, type checks, and the full test suite.
- Refactor for clarity and remove duplication introduced by this feature.
- Remove imports/variables/functions that YOUR changes made unused. Don't
  delete pre-existing dead code — mention it instead.
- Confirm the spec, tests, and implementation still agree.

**Gate:** Full suite, build, and linters pass. Behavior is unchanged from the
end of Phase 4 (tests still green).

---

## Phase 6 — Complete & Squash Commit

**Goal:** Ship a clean, reviewable unit of work.

- Verify the feature end-to-end against the Phase 1 success criteria.
- Clean up temporary files and scaffolding created during development.
- Squash the work into a single, coherent commit with a clear message
  (what and why). Only commit when explicitly asked.
- Stage specific files rather than everything; flag any files that may contain
  secrets before committing.

**Gate:** Success criteria met, working tree clean, and a single squashed
commit ready with a descriptive message.