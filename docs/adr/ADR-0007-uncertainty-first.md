# ADR-0007: Uncertainty-first segmentation

**Status:** Accepted  
**Date:** 2026-06

## Decision
Every segmentation label carries a per-cell uncertainty score, computed from distance to the nearest speed-threshold boundary.

## Formula
```
uncertainty = exp(−margin / margin_scale)
```
where `margin` is the distance (m/s) from the reconstructed speed to the nearest class boundary.

## Rationale
- Cells deep inside a tissue class (far from any boundary) have uncertainty ≈ 0
- Cells on a boundary have uncertainty ≈ 1
- This is explainable: a human (or downstream system) can see exactly why uncertainty is high

## Consequences
- UI must render uncertainty overlay when displaying segmentation
- Confidence routing uses per-cell uncertainty to decide when human review is warranted
- No neural network required; the model is fully interpretable
