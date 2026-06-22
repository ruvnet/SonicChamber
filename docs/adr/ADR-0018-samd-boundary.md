# ADR-0018: SaMD regulatory boundary

**Status:** Accepted  
**Date:** 2026-06

## Decision
MetaBioHacker operates below the Software as a Medical Device (SaMD) regulatory threshold unless explicitly deployed for clinical decision support.

## Permitted modes
| Mode | Scope |
|------|-------|
| `research` | Dataset construction, physics validation, model evaluation |
| `wellness` | General, non-diagnostic self-baseline monitoring |
| `clinicalReview` | Software proposes, licensed professional decides |

## Prohibited without regulatory pathway
- Influencing diagnosis, treatment, triage, or clinical management
- Presenting results without uncertainty overlay
- Suppressing the human-review flag for pathology/biopsy observations

## Regulatory references
- FDA Good Machine Learning Practice (GMLP)
- FDA Predetermined Change Control Plans (PCCP)
- Ontario PHIPA (Personal Health Information Protection Act)

## Enforcement
The governance gate `pathology → humanReviewRequired = true` is a CI-enforced invariant. The `allowedOutputMode` field in every run ledger gates what the UI may display.
