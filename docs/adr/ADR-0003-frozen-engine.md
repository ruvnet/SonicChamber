# ADR-0003: Frozen physics engine, evolved harness

**Status:** Accepted  
**Date:** 2026-06

## Decision
`sonic-ct` is immutable once released. The MetaBioHacker harness (policy, routing, evidence) evolves freely on top.

## Rationale
The acoustic residual is the ground truth. Anything downstream (prior fusion, evidence grading, model routing) must not alter it. Making the engine frozen enforces this structurally, not just by convention.

## Implementation
- `sonic_ct_serve` reads JSON policy on stdin, runs the frozen engine, returns scores
- Darwin harness mutations operate only on policy/routing parameters
- The governance gate asserts `score.acousticResidual == engine.acousticResidual` in CI

## Consequences
- Harness improvements can be shipped without touching the physics or its test suite
- Any physics change requires a full physics test re-run and a new ADR
