# Governance & Safety

## Scope

MetaBioHacker is research infrastructure, not a medical device. Three operating modes are defined:

| Mode | What it enables | What is prohibited |
|------|----------------|-------------------|
| `research` | Dataset construction, physics benchmarking, model evaluation | Presenting to patients |
| `wellness` | Non-diagnostic self-baseline monitoring | Clinical language, diagnosis-adjacent claims |
| `clinicalReview` | Decision support reviewed by a licensed professional | Autonomous diagnosis, treatment decisions |

## Four CI-enforced invariants

These gates fail the build if violated:

### 1. Acoustic residual is immutable
Downstream context (prior fusion, contradiction penalty, evidence grading) must never alter the frozen engine's acoustic output.

```typescript
// In CI gate:
expect(multimodalScore.acousticResidual).toBe(acousticScore.acousticResidual);
```

### 2. Pathology forces human review
Any observation with `modality ∈ {pathology, biopsy, pap, hpv, cytology}` must set `humanReviewRequired = true`.

### 3. Evidence grading gate
Only claims with Grade A or Grade B evidence AND at least one citation may pass `gateEvidence()`. Grade C/D claims are blocked.

### 4. Honesty gate
Reconstruction results with mean Dice < 0.4 are labelled `aboveHonestyGate = false` and must not be surfaced as headline performance numbers.

## Run ledger

Every pipeline execution produces a `ReconstructionLedger` with stable SHA-256 hashes of:
- All input observations (canonical form)
- The reconstruction prior
- The patient state graph
- The multimodal score
- The ledger itself (chain hash)

This enables third-party verification of any published result.

## Standards compliance

| Standard | Role |
|----------|------|
| DICOM | Medical imaging containers |
| HL7 FHIR | Clinical data exchange |
| LOINC | Lab observation codes |
| SNOMED CT | Clinical terminology |
| OMOP CDM | Research analytics |
| FDA GMLP | Good Machine Learning Practice guidance |
| FDA PCCP | Predetermined Change Control Plans |
| PHIPA (Ontario) | Personal health information privacy |

## Human review path

Any run where `humanReviewRequired = true`:
1. Output is flagged with `allowedOutputMode: "clinicalReview"`
2. UI must display uncertainty overlay
3. Diagnostic language is blocked in the output packet
4. The ledger records which observation triggered the flag and why
