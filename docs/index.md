# MetaBioHacker Documentation

## Guides

- [Quick Start](../README.md#quick-start)
- [Physics Reference](PHYSICS.md) — USCT, SART, FWI, tissue acoustics, honest limitations
- [Governance & Safety](GOVERNANCE.md) — CI invariants, evidence grading, SaMD scope
- [WASM API Reference](api/sonic-ct-wasm.md) — C ABI, zero-copy buffer access

## Architecture Decisions

| ADR | Topic |
|-----|-------|
| [ADR-0001](adr/ADR-0001-rust-zero-dep-core.md) | Zero-dependency Rust engine |
| [ADR-0003](adr/ADR-0003-frozen-engine.md) | Frozen physics, evolved harness |
| [ADR-0007](adr/ADR-0007-uncertainty-first.md) | Uncertainty-first segmentation |
| [ADR-0009](adr/ADR-0009-five-tissue-classes.md) | Five acoustic classes only |
| [ADR-0018](adr/ADR-0018-samd-boundary.md) | SaMD regulatory boundary |

## Key numbers

| Metric | Value | Condition |
|--------|-------|-----------|
| Reconstruction time | ~130 ms | WASM, 96×96, 256 elements, 8 SART iters |
| WASM binary size | 31 KB | No wasm-bindgen, C ABI only |
| Fat Dice | ~0.72 | Synthetic phantom, tuned SegModel |
| Muscle Dice | ~0.65 | Synthetic phantom |
| Bone Dice | ~0 | Expected — straight-ray physics; FWI roadmap |
| Mean Dice (5 classes) | ~0.63 | Tuned model vs. 0.30 default |
| Insert throughput (memory) | 150K/s | NSW acoustic index |
