# ADR-0009: Five acoustic tissue classes — no organ identity from speed

**Status:** Accepted  
**Date:** 2026-06

## Decision
Speed-of-sound maps are segmented into exactly five acoustic classes:
Water · Fat · Muscle · Organ (soft parenchyma) · Bone

Organ identity (liver vs. spleen vs. kidney) is **never** derived from speed alone.

## Rationale
Liver (1579 m/s) and spleen (1567 m/s) have overlapping speed distributions. Misidentifying tissue type from speed would be a clinically dangerous hallucination.

Organ identity is inferred in a separate layer (`organ.rs`) using:
- Shape and size (template matching)
- Anatomical position (prior knowledge)
- Consistency across slices in a volume sweep

## Consequences
- Segmentation output is physically honest and auditable
- Organ inference is probabilistic and carries a confidence score + evidence bitmask
- No code path maps a speed value directly to an organ name
