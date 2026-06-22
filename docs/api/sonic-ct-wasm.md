# sonic-ct-wasm C ABI Reference

The WASM binding layer exports a minimal C ABI — no wasm-bindgen, no JavaScript glue beyond what you write yourself.

## Loading the module (Node)

```javascript
import { readFileSync } from 'node:fs';
import { fileURLToPath } from 'node:url';
import * as path from 'node:path';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const wasmBytes = readFileSync(path.join(__dirname, 'sonic_ct_bg.wasm'));
const { instance } = await WebAssembly.instantiate(wasmBytes, {});
const wasm = instance.exports;
```

## 2-D (single slice) API

### `sct_run(n, elements, fan, iters, seed) → i32`
Run a complete 2-D reconstruction.
- `n` — grid side length (e.g. 96)
- `elements` — number of ring transducers (e.g. 256)
- `fan` — active receive fan angle in degrees (e.g. 270)
- `iters` — SART iterations (1 = backprojection, 8 = default)
- `seed` — phantom random seed (same seed = same phantom)
- Returns `1` on success, `0` on error

### Scalar getters (call after `sct_run`)
| Function | Return | Description |
|---|---|---|
| `sct_mae()` | `f32` | Mean absolute speed error (m/s) |
| `sct_mean_dice()` | `f32` | Mean Dice across tissue classes |
| `sct_measurements()` | `u32` | Valid ray count |
| `sct_dice(class)` | `f32` | Per-class Dice (0=water..4=bone) |
| `sct_anomaly()` | `u32` | 1 if anatomical anomaly detected |

### Buffer pointer getters
Read flat F32 arrays directly from WASM linear memory — zero copy.

```javascript
const n = wasm.sct_grid_n();
const ptr = wasm.sct_recon_speed_ptr() / 4;  // divide by 4 for Float32Array index
const speed = new Float32Array(wasm.memory.buffer).slice(ptr, ptr + n * n);
```

| Function | Type | Layout |
|---|---|---|
| `sct_ring_xy_ptr()` | `*f32` | `[x0,y0, x1,y1, ...]` — element count × 2 |
| `sct_truth_speed_ptr()` | `*f32` | `n×n` row-major speed grid (m/s) |
| `sct_recon_speed_ptr()` | `*f32` | `n×n` reconstructed speed |
| `sct_recon_atten_ptr()` | `*f32` | `n×n` reconstructed attenuation (Np/m) |
| `sct_truth_labels_ptr()` | `*u8` | `n×n` ground truth tissue labels |
| `sct_recon_labels_ptr()` | `*u8` | `n×n` reconstructed tissue labels |
| `sct_uncertainty_ptr()` | `*f32` | `n×n` per-cell segmentation uncertainty |

## 3-D (progressive volume) API

### `sct_vol_begin(nz, n, elements, fan, iters, seed) → i32`
Begin a multi-slice volume sweep.
- `nz` — number of axial slices

### `sct_vol_step() → i32`
Reconstruct the next slice. Call once per animation frame.
- Returns number of slices completed so far
- When return value equals `nz`, sweep is complete and organ inference runs

### Volume getters
| Function | Return | Description |
|---|---|---|
| `sct_vol_cursor()` | `u32` | Current slice index |
| `sct_vol_mean_dice()` | `f32` | Running mean Dice |
| `sct_vol_confidence()` | `f32` | Overall volume confidence |
| `sct_vol_fraction(c)` | `f32` | Fraction of volume classified as class `c` |
| `sct_organ_count()` | `u32` | Hypothesized organs (after sweep completes) |
| `sct_organ_id(i)` | `u32` | Organ semantic ID |
| `sct_organ_conf(i)` | `f32` | Hypothesis confidence 0–1 |
| `sct_quality_flag(f)` | `u32` | Severity: 0=bone shadow, 1=sparse, 2=boundary, 3=gas |
