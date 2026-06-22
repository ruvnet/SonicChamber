# MetaBioHacker

> Ultrasound Computed Tomography simulator + multimodal medical signal fusion harness.
> Rust physics engine · WebAssembly runtime · TypeScript SDK · zero patient data.

[![CI](https://github.com/ruvnet/MetaBioHacker/actions/workflows/ci.yml/badge.svg)](https://github.com/ruvnet/MetaBioHacker/actions/workflows/ci.yml)
[![Docs](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://ruvnet.github.io/MetaBioHacker)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## What Is This?

MetaBioHacker is a **research platform** for Ultrasound Computed Tomography (USCT) — a non-ionising medical imaging modality that reconstructs the speed-of-sound and attenuation through tissue cross-sections using a ring of ultrasound transducers.

The system is composed of three layers:

```
┌──────────────────────────────────────────────────────────────────┐
│  FROZEN PHYSICS ENGINE  (crates/sonic-ct)                        │
│  SART reconstruction · FWI · Segmentation · Acoustic Memory      │
│  Zero external deps · Deterministic · WASM-native                │
├──────────────────────────────────────────────────────────────────┤
│  WASM BINDING LAYER  (crates/sonic-ct-wasm)                      │
│  C ABI exports · zero-copy buffer passing · 31 KB binary         │
├──────────────────────────────────────────────────────────────────┤
│  FUSION HARNESS  (packages/metabiohacker)                        │
│  Multimodal ingest · Patient state graph · Prior builder         │
│  Evidence gate · Run ledger · Darwin harness evolution           │
└──────────────────────────────────────────────────────────────────┘
```

**Design principle:** the physics engine is frozen and immutable; only the harness (policy, routing, evidence grading) evolves. The engine's acoustic residual is never modified by downstream context.

---

## What USCT Does

A ring of ~256 ultrasound transducers surrounds a subject (currently: a procedurally-generated digital human cross-section). Each transducer fires in turn; all others listen. From the first-arrival travel times, the system solves a tomographic inverse problem to recover a speed-of-sound map.

```
Ring acquisition:
  Each element fires → N-1 receivers measure travel time
  Travel time ∝ ∫(1/c(x,y)) ds  (line integral of slowness)

Reconstruction:
  Solve  A·s = t  (SART: Simultaneous Algebraic Reconstruction Technique)
  A_ij = length ray i spends in cell j
  s    = per-cell slowness (1/c)
  t    = measured travel times

Tissue classification:
  Speed → {Water, Fat, Muscle, Organ, Bone}  (five acoustic classes)
  Uncertainty = exp(−margin_to_boundary / scale)
```

Typical reconstruction: **~130 ms** in WebAssembly on a 96×96 grid with 256 elements and 8 SART iterations.

### Honest limitations

- Straight-ray physics: diffraction and refraction ignored → bone reconstruction is poor (Dice ≈ 0)
- Synthetic phantom only: no real patient data in this codebase
- Full-Waveform Inversion (FWI) is implemented and tested but is the research frontier, not the default

---

## Quick Start

### Native (Rust CLI)

```bash
git clone https://github.com/ruvnet/MetaBioHacker
cd MetaBioHacker

# Run reconstruction demo → PGM images + metrics
cargo run --release --bin sonic_ct_demo /tmp/out

# Train segmentation model on synthetic corpus
cargo run --release --bin sonic_ct_train 24 /tmp/out

# Benchmark reconstruction methods (BP vs SART vs Landweber)
cargo run --release --bin sonic_ct_methods
```

### WebAssembly (browser demo)

```bash
# Build WASM
bash scripts/build-sonic-ct-wasm.sh

# Run React Three Fiber demo
cd examples/sonic-ct
npm install && npm run dev
# → http://localhost:5184
```

### TypeScript / Node

```bash
npm install metabiohacker

# or from source:
cd packages/metabiohacker && npm install
```

```typescript
import { buildReconstructionPrior, scoreMultimodalRun } from 'metabiohacker';
import type { MedicalObservation } from 'metabiohacker';

const obs: MedicalObservation[] = [
  { modality: 'mri', uncertainty: 0.1, /* ... */ },
  { modality: 'lab', uncertainty: 0.05, /* ... */ },
];

const prior = buildReconstructionPrior(obs);
// prior.anatomyPriorWeight → 0.9 (from MRI)
// prior.humanReviewRequired → false (no pathology)

const score = scoreMultimodalRun({ acoustic: engineResult, prior });
// score.acousticResidual → UNCHANGED (frozen engine output)
// score.reconstructionStability → acoustic + ~10–12% from priors
```

---

## Architecture

### Physics Engine (`crates/sonic-ct`)

| Module | Purpose |
|--------|---------|
| `phantom.rs` | Procedural digital human: fat ring, muscle layer, organs, spine |
| `ring.rs` | Transducer ring geometry at configurable radius |
| `acquisition.rs` | Ray tracing, travel-time + attenuation simulation |
| `reconstruction.rs` | SART, Backprojection, Landweber, configurable iterations |
| `segmentation.rs` | Speed → tissue class + per-cell uncertainty |
| `metrics.rs` | Dice, MAE, RMSE, PSNR, SSIM vs. ground truth |
| `fwi.rs` | Full-Waveform Inversion: wave equation, adjoint gradient, multiscale |
| `memory.rs` | Acoustic NSW index: nearest-neighbour patient matching, FWI warm start |
| `organ.rs` | Post-hoc organ hypothesis from shape + position (not speed alone) |

### WASM Binding Layer (`crates/sonic-ct-wasm`)

C ABI exports — no wasm-bindgen, 31 KB binary, zero-copy buffer access from JS:

```c
// 2-D single-slice
sct_run(n, elements, fan, iters, seed) → 1 (ok) / 0 (err)
sct_mae()          → f32   // mean absolute speed error
sct_mean_dice()    → f32   // mean tissue Dice across classes
sct_recon_speed_ptr() → *f32[n*n]   // read flat buffer from WASM memory

// 3-D progressive volume sweep
sct_vol_begin(nz, n, elements, fan, iters, seed) → 1 (ok) / 0 (err)
sct_vol_step() → slices_completed    // call once per animation frame
sct_vol_fraction(class) → f32        // body composition (0=water .. 4=bone)
sct_organ_count()      → u32         // hypothesized organs after sweep
sct_organ_id(i)        → u32         // organ semantic label
sct_organ_conf(i)      → f32         // hypothesis confidence
```

### Fusion Harness (`packages/metabiohacker`)

```
MedicalObservation (typed, provenance, uncertainty, consent scope)
        ↓
Patient State Graph (nodes + edges, temporal ordering, contradiction detection)
        ↓
Prior Builder (obs → prior weights; pathology → human review forced)
        ↓
Scoring (prior lifts stability ~10–12%; acousticResidual is immutable)
        ↓
Evidence Gate (Grade A/B only; C/D blocked; citations required)
        ↓
Run Ledger (stable hashes, routing decisions, safety flags)
```

Supported modalities: `acoustic · mri · ct · ultrasound · ekg · eeg · lab · pathology · cytology · pap · hpv · biopsy · wearable`

---

## CI Guards

```yaml
# .github/workflows/ci.yml  (triggers on every push/PR)

rust-engine:
  ✓ cargo test --release       # 16 physics tests
  ✓ cargo clippy --lib         # zero warnings
  ✓ cargo fmt --check          # formatting gate
  ✓ build WASM artifact        # scripts/build-sonic-ct-wasm.sh

metabiohacker-gates:
  ✓ npm test                   # 14 governance tests

github-pages:
  ✓ deploy docs/               # on push to main
```

### Governance gates (must pass before merge)

| Gate | What it enforces |
|------|-----------------|
| Acoustic residual invariant | Contradiction penalty never mutates the frozen engine's output |
| Pathology → human review | Pathology/biopsy/cytology modalities always set `humanReviewRequired = true` |
| Evidence grading | Claims with Grade C/D or no citations are blocked by the evidence gate |
| Honesty gate | Reconstruction results below Dice 0.4 are not presented as headline numbers |

---

## Medical Standards

| Standard | Used for |
|----------|---------|
| DICOM | Imaging containers, pixel data, study/series structure |
| HL7 FHIR | Clinical data exchange, observation resources |
| LOINC | Lab test codes in `MedicalObservation.code` |
| SNOMED CT | Clinical findings in the patient state graph |
| OMOP CDM | Research-scale observational analytics |

---

## Scope & Governance

MetaBioHacker operates in **three modes**, none of which constitute autonomous diagnosis:

| Mode | Scope |
|------|-------|
| `research` | Dataset construction, model evaluation, physics validation |
| `wellness` | General, non-diagnostic self-baseline monitoring |
| `clinicalReview` | Software proposes, licensed professional decides |

Any use that influences diagnosis, treatment, triage, or management is subject to regulatory pathways (FDA GMLP / PCCP, PHIPA). See [ADR-0018](docs/adr/ADR-0018-samd-boundary.md) and [ADR-0019](docs/adr/ADR-0019-medical-signal-os.md).

**No real patient data is included in this repository.**

---

## Architecture Decisions

| ADR | Decision |
|-----|---------|
| [ADR-0001](docs/adr/ADR-0001-rust-zero-dep-core.md) | Rust zero-dependency physics engine |
| [ADR-0002](docs/adr/ADR-0002-wasm-c-abi.md) | C ABI WASM bindings (no wasm-bindgen) |
| [ADR-0003](docs/adr/ADR-0003-frozen-engine.md) | Frozen physics, evolved harness |
| [ADR-0004](docs/adr/ADR-0004-sart-baseline.md) | SART as reconstruction baseline |
| [ADR-0007](docs/adr/ADR-0007-uncertainty-first.md) | Uncertainty-first segmentation |
| [ADR-0009](docs/adr/ADR-0009-five-tissue-classes.md) | Five acoustic classes, no organ labels from speed |
| [ADR-0014](docs/adr/ADR-0014-darwin-harness.md) | Darwin multi-objective harness evolution |
| [ADR-0016](docs/adr/ADR-0016-medical-standards.md) | DICOM/FHIR/LOINC/SNOMED alignment |
| [ADR-0018](docs/adr/ADR-0018-samd-boundary.md) | SaMD regulatory boundary |
| [ADR-0019](docs/adr/ADR-0019-medical-signal-os.md) | Medical Signal Operating System architecture |
| [ADR-0026](docs/adr/ADR-0026-fwi-roadmap.md) | Full-Waveform Inversion roadmap |

---

## Contributing

1. Fork and open a PR against `main`
2. All CI guards must pass — specifically the four governance gates
3. New reconstruction changes require a physics test (Dice or MAE regression threshold)
4. New harness changes require a governance test in `packages/metabiohacker/test/`
5. Significant decisions → new ADR in `docs/adr/`

---

## License

MIT — see [LICENSE](LICENSE).

---

*Born from [ruvnet/ruvector](https://github.com/ruvnet/ruvector) PR #595 · June 2026*
