# Physics Reference

## Ultrasound Computed Tomography (USCT)

USCT recovers the speed-of-sound `c(x,y)` and attenuation `α(x,y)` distribution inside a subject by firing ultrasound pulses from a ring of transducers and measuring how long each pulse takes to arrive at every receiver.

### Forward model (straight-ray approximation)

For a ray from transmitter `s` to receiver `r`, the travel time is:

```
t_sr = ∫_path (1 / c(x,y)) ds ≈ Σ_j  A_{ij} · (1/c_j)
```

where `A_{ij}` is the length of ray `i` through grid cell `j` and `c_j` is the speed in that cell. This is the straight-ray (first-arrival) approximation — refraction and diffraction are ignored.

### Tissue acoustic properties

| Tissue | Speed (m/s) | Attenuation (Np/m) |
|--------|------------|-------------------|
| Water (coupling bath) | 1480 | 0.5 |
| Fat | 1450 | 8 |
| Muscle | 1580 | 15 |
| Soft organ (parenchyma) | 1570 | 12 |
| Cortical bone | 3000 | 120 |

### SART reconstruction

Solves `A·s = t` (slowness `s = 1/c`) by the row-action update:

```
For each ray i:
  residual  = (t_i − Σ_j A_{ij}·s_j) / Σ_j A_{ij}
  s_j      += relaxation × A_{ij} × residual   for all j on ray i
```

One sweep over all rays = one SART iteration. Multiple sweeps converge toward the least-squares solution. Default: 8 iterations, relaxation 0.9.

**One iteration** (Δiters=1) is equivalent to delay-and-sum backprojection.

### Segmentation uncertainty

```
uncertainty(cell) = exp(−margin / scale)
```

`margin` = distance (m/s) between the reconstructed speed and the nearest class boundary.  
`scale` = 30 m/s (default).

Cells deep inside a class → uncertainty ≈ 0.  
Cells near a boundary → uncertainty ≈ 1.

### Full-Waveform Inversion (FWI)

FWI solves the 2-D scalar acoustic wave equation:

```
∂²p/∂t² = c²(x,y) ∇²p + f(x,t)
```

The gradient of the data-misfit χ with respect to the squared-slowness κ = c² is:

```
∂χ/∂κ = Σ_t  λ(x,t) · ∇²p(x,t)
```

where `λ` is the adjoint (back-propagated residual) wavefield. This is solved iteratively with frequency continuation: low-frequency stages first (smooth model, cycle-skip-robust), then progressively higher frequencies.

FWI is the roadmap item for sub-mm resolution (ADR-0026). It is implemented and tested but not the default reconstruction path.

### Known limitations

| Limitation | Cause | Planned fix |
|-----------|-------|------------|
| Bone Dice ≈ 0 | Straight rays cannot model acoustic shadow | FWI (ADR-0026) |
| Soft organ Dice ≈ 0.3–0.4 | Speed overlap between liver/spleen/kidney | Multi-frequency FWI |
| Synthetic phantom only | No real-data pipeline yet | DICOM/FHIR ingest (ADR-0016) |
| 2-D only (default) | 3-D volume is slice-by-slice, not truly 3-D | True 3-D wave equation (future) |
