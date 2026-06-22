# ADR-0001: Rust zero-dependency physics engine

**Status:** Accepted  
**Date:** 2026-06

## Decision
The core simulation (`sonic-ct`) uses only Rust `std` — no external crates.

## Rationale
- WASM target: every external crate risks adding incompatible syscalls or std features that break `wasm32-unknown-unknown`
- Auditability: a reviewer can trace every calculation without chasing transitive deps
- Build reproducibility: zero `Cargo.lock` churn from upstream releases

## Consequences
- NSW graph, PRNG, geometry, metrics all hand-written
- Build time: < 5 s (no proc-macros, no heavy crates)
- WASM binary: 31 KB (vs. ~1 MB with wasm-bindgen + serde)
