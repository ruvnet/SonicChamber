# Sonic Chamber

> **The open-source Acoustic Digital Human Workbench.**  
> Full-body ultrasound computed tomography simulation, 3D reconstruction,
> multimodal medical signal fusion, and autonomous harness evolution — in your browser.

[![CI](https://github.com/ruvnet/SonicChamber/actions/workflows/ci.yml/badge.svg)](https://github.com/ruvnet/SonicChamber/actions/workflows/ci.yml)
[![Live Demo](https://img.shields.io/badge/demo-live%20%E2%86%92-brightgreen)](https://ruvnet.github.io/SonicChamber/demo/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/Rust-zero--dep-orange)](crates/sonic-ct)
[![WASM](https://img.shields.io/badge/WASM-31%20KB-blue)](crates/sonic-ct-wasm)

---

## Live Demo

**[→ Open Sonic Chamber in your browser](https://ruvnet.github.io/SonicChamber/demo/)**  
No install. Full USCT reconstruction running in WebAssembly (~130 ms).

[![Sonic Chamber — Acoustic Digital Human Workbench](docs/assets/screenshot.png)](https://ruvnet.github.io/SonicChamber/demo/)

---

## What Is Sonic Chamber?

Sonic Chamber is an **open-source research workbench** for Ultrasound Computed Tomography (USCT) — a non-ionising medical imaging modality that reconstructs speed-of-sound and attenuation maps through full-body tissue cross-sections.

It is the open-source reference implementation for the type of acoustic digital human modelling pioneered by emerging platforms like **Midjourney Medical**. Where those platforms are closed, Sonic Chamber is fully auditable — every algorithm, every weight, every ADR is public.

### Sonic Chamber vs. Midjourney Medical

Midjourney Medical applies generative AI to medical imaging — synthesising, enhancing, and reconstructing diagnostic images from multimodal inputs (CT, MRI, ultrasound, labs). Sonic Chamber is the acoustic physics layer of that stack, built open and reproducible:

| Midjourney Medical | Sonic Chamber |
|---|---|
| Proprietary cloud platform | MIT open source — runs locally + in-browser |
| Black-box reconstruction | Every algorithm documented, ADR-indexed |
| No audit trail | Cryptographic run ledgers (SHA-256 chain) |
| Closed training data | Synthetic phantom + DICOM/FHIR adapters |
| Subscription model | Free forever |

---

## Hardware Parts List

Sonic Chamber works as pure software simulation or as the software stack for a real USCT rig.

### Software-Only (recommended starting point)

| Component | Spec | Cost |
|-----------|------|------|
| Any modern PC / Mac / Linux | 4-core CPU, 8 GB RAM | — |
| Browser with WASM support | Chrome / Firefox / Safari | Free |
| Rust toolchain | stable 1.78+ | Free |
| Node.js | v22+ | Free |

### Entry-Level Hardware Rig (~$3–12K)

| Component | Part | Qty | Est. Cost |
|-----------|------|-----|-----------|
| Controller SBC | Raspberry Pi 5 (8 GB) | 1 | ~$80 |
| Ultrasound transducer array | 128-element ring, 2–5 MHz | 1 | $800–$3,000 |
| Pulser/receiver board | Ultrasonix SonixDAQ or equivalent | 1 | $2,000–$8,000 |
| Water tank (coupling bath) | 30 cm acrylic cylinder, food-grade | 1 | ~$150 |
| Stepper motor (axial drive) | NEMA 17, 1.8°/step + A4988 driver | 1 | ~$25 |
| ADC / FPGA interface | 12-bit, 100 MSPS min | 1 | $200–$500 |
| Coaxial cables + SMA connectors | RG-58, SMA-M | — | ~$50 |
| **Total** | | | **~$3,300–$12,000** |

### Research-Grade Rig (~$108–200K)

| Component | Part | Qty | Est. Cost |
|-----------|------|-----|-----------|
| Ultrasound research platform | Verasonics Vantage 256 or equivalent | 1 | $80K–$150K |
| 256-element ring transducer | Imasonic custom, 2–7 MHz | 1 | $15K–$40K |
| High-perf workstation | 32-core, 128 GB RAM, RTX 4090 | 1 | ~$8,000 |
| Degassed water recirculation | Temperature-controlled DI water system | 1 | ~$2,000 |
| Precision stepper + rail | 5-axis stage, sub-mm positioning | 1 | ~$3,000 |
| **Total** | | | **~$108,000–$203,000** |

### Open-Source Hardware Alternatives

| Platform | Channels | Open HW | Est. Cost |
|----------|---------|---------|-----------|
| OpenSonics | 64 | Yes | ~$500 |
| ULA-OP | 64 | Partial | ~$15,000 |
| Kwave (simulation only) | N/A | Yes | Free |
| FIELD II (simulation only) | N/A | Yes | Free |

> **Design first, buy later:** Sonic Chamber's simulation produces the same reconstruction behaviour as hardware. Design and validate your acquisition protocol in software before committing to hardware spend.

---

## Setup Instructions

### Option 1 — Browser (zero install)

**[→ ruvnet.github.io/SonicChamber/demo/](https://ruvnet.github.io/SonicChamber/demo/)**

### Option 2 — Local (Rust + WASM + React)

```bash
# Install Rust + wasm-pack
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Clone and run
git clone https://github.com/ruvnet/SonicChamber
cd SonicChamber
bash scripts/build-sonic-ct-wasm.sh
cd examples/sonic-ct && npm install && npm run dev
# → http://localhost:5184
```

### Option 3 — Native CLI (fastest, no WASM overhead)

```bash
cd SonicChamber

# One reconstruction → PGM images + metrics to stdout
cargo run --release --bin sonic_ct_demo /tmp/out

# Train segmentation model on 24 synthetic phantoms
cargo run --release --bin sonic_ct_train 24 /tmp/out

# Benchmark reconstruction methods
cargo run --release --bin sonic_ct_methods

# Frozen-engine JSON stdio (for ML / harness integration)
echo '{"n":96,"elements":256,"iters":8,"seed":42}' \
  | cargo run --release --bin sonic_ct_serve
```

### Option 4 — Docker

```bash
docker run --rm -p 5184:5184 ghcr.io/ruvnet/sonic-chamber:latest
# → http://localhost:5184
```

### Option 5 — TypeScript / Node.js

```bash
npm install sonic-chamber
```

```typescript
import { buildReconstructionPrior, scoreMultimodalRun } from 'sonic-chamber';

const prior = buildReconstructionPrior([
  { modality: 'mri',  uncertainty: 0.1,  consentScope: 'research' },
  { modality: 'ekg',  uncertainty: 0.05, consentScope: 'research' },
]);

const score = scoreMultimodalRun({ acoustic: engineResult, prior });
// score.acousticResidual  → immutable physics output (never modified)
// score.reconstructionStability → +10–12% from multimodal prior fusion
```

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│  FROZEN PHYSICS ENGINE  (crates/sonic-ct)                            │
│  SART · Backprojection · Landweber · FWI · Acoustic Memory · Organs  │
│  Zero external Rust deps · Deterministic · 31 KB WASM binary         │
├──────────────────────────────────────────────────────────────────────┤
│  WASM BINDING LAYER  (crates/sonic-ct-wasm)                          │
│  C ABI · Zero-copy F32/U8 buffer access · 2-D + 3-D volume API       │
├──────────────────────────────────────────────────────────────────────┤
│  FUSION HARNESS  (packages/metabiohacker)                            │
│  Multimodal ingest · Patient state graph · Prior builder             │
│  Darwin evolution · Evidence gate · Immutable run ledger             │
└──────────────────────────────────────────────────────────────────────┘
```

The physics engine is **frozen** — its acoustic residual is never altered by downstream context.

---

## Features

### Reconstruction Methods

| Method | Description | WASM time |
|--------|-------------|-----------|
| `Backprojection` | Delay-and-sum (1 SART iteration) | ~16 ms |
| `SART` | Algebraic row-action, 8 iterations default | ~130 ms |
| `Landweber` | Gradient descent on normal equations | ~200 ms |
| `FWI` | Full-Waveform Inversion, adjoint-state gradient | ~500 ms/src |

### Tissue Classification

| Tissue | Speed | Attenuation | Dice (tuned model) |
|--------|-------|-------------|-------------------|
| Water (coupling bath) | 1480 m/s | 0.5 Np/m | — |
| Fat | 1450 m/s | 8 Np/m | ~0.72 |
| Muscle | 1580 m/s | 15 Np/m | ~0.65 |
| Organ (soft parenchyma) | 1570 m/s | 12 Np/m | ~0.41 |
| Bone (cortical) | 3000 m/s | 120 Np/m | ~0.00 ¹ |

> ¹ Bone Dice ≈ 0 with straight-ray SART (expected — acoustic shadow ignored). Full-Waveform Inversion (ADR-0026) is the path to sub-mm bone resolution.

### Additional Capabilities

- **3-D volume sweep** — progressive axial slicing, body composition fractions
- **Organ inference** — hypothesis from shape + position, never speed alone
- **Acoustic memory** — NSW graph for longitudinal tracking and FWI warm-start
- **Full-Waveform Inversion** — adjoint-state, frequency continuation, CI gradient-checked
- **Darwin harness evolution** — Pareto-front multi-objective parameter search
- **Run ledger** — SHA-256 chain hash for every pipeline execution
- **Evidence gate** — Grade A/B-only claims; Grade C/D blocked
- **Multimodal fusion** — MRI · CT · EKG · EEG · labs · pathology priors

---

## CI Gates (non-negotiable)

| Gate | What it enforces |
|------|-----------------|
| Acoustic residual immutable | Physics output never altered by downstream fusion |
| Pathology → human review | Pathology/biopsy always sets `humanReviewRequired = true` |
| Evidence grading | Grade C/D or no-citation claims blocked |
| Honesty gate | Results below Dice 0.4 not surfaced as headline numbers |

---

## Scope & Safety

| Mode | Scope |
|------|-------|
| `research` | Dataset construction, physics benchmarking, model evaluation |
| `wellness` | Non-diagnostic self-baseline monitoring |
| `clinicalReview` | Decision support — licensed professional decides |

Clinical use requires a regulatory pathway (FDA GMLP / PCCP, PHIPA).  
**No real patient data is included in this repository.**

---

## Architecture Decisions

| ADR | Decision |
|-----|---------|
| [ADR-0001](docs/adr/ADR-0001-rust-zero-dep-core.md) | Zero-dependency Rust engine |
| [ADR-0002](docs/adr/ADR-0002-wasm-c-abi.md) | C ABI WASM, 31 KB |
| [ADR-0003](docs/adr/ADR-0003-frozen-engine.md) | Frozen physics, evolved harness |
| [ADR-0004](docs/adr/ADR-0004-sart-baseline.md) | SART as default reconstruction |
| [ADR-0007](docs/adr/ADR-0007-uncertainty-first.md) | Uncertainty-first segmentation |
| [ADR-0009](docs/adr/ADR-0009-five-tissue-classes.md) | Five acoustic classes only |
| [ADR-0014](docs/adr/ADR-0014-darwin-harness.md) | Darwin multi-objective evolution |
| [ADR-0018](docs/adr/ADR-0018-samd-boundary.md) | SaMD regulatory boundary |
| [ADR-0019](docs/adr/ADR-0019-medical-signal-os.md) | Medical Signal OS architecture |
| [ADR-0026](docs/adr/ADR-0026-fwi-roadmap.md) | FWI roadmap for sub-mm resolution |

---

## Contributing

1. Fork → branch → PR against `main`
2. All four CI governance gates must pass
3. New reconstruction methods require Dice / MAE regression tests
4. New harness changes require a governance test in `packages/metabiohacker/test/`
5. Significant architectural decisions → new ADR in `docs/adr/`

---

## License

MIT — see [LICENSE](LICENSE).

---

*Sonic Chamber · Open-source acoustic digital human workbench · [ruvnet/SonicChamber](https://github.com/ruvnet/SonicChamber)*  
*Born from [ruvnet/ruvector](https://github.com/ruvnet/ruvector) PR #595 · June 2026*
