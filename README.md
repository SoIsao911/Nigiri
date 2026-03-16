# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.3

Browser-based formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing. Single-file HTML — no server required.

## Core Workflow: Diagnose → Prescribe → Verify

1. **Diagnose** (Quality Engine): Identifies weakest quality dimensions + explains WHY
2. **Prescribe** (PSO Optimizer): Factory-safe parameter suggestions (5g/0.2°C steps)
3. **Verify** (Monte Carlo): 10,000 simulations at your actual process precision

## Quality Engine

Six-edge tetrahedron model. QI = GeoMean × Balance Factor.

| Edge | Meaning | Model | Adaptive |
|------|---------|-------|----------|
| e₁ F─T | Temperature match | Resin-specific linear + KNN | ✓ |
| e₂ F─Ti | Color MB interference | **Pigment-class-weighted Gaussian** | ✓ |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell (penalizes over-reaction) | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | Context-aware f(Temp1) + KNN | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### Context-Aware DCP Supply (e₅)

```
supply_optimal = -1.90 × Temp1 + 395.3
```

Low Temp1 → need more DCP → higher supply is normal. Combined with KNN for family-specific patterns.

### Data-Driven Pigment Absorption (e₂)

Coefficients validated on 244 production records + 179 BOM formulations with known Pigment Index.

**DCP Absorption** (per phr pure pigment):

| Pigment Class | Coefficient | MB Examples | Chemical Basis |
|:-------------|:-----------|:------------|:---------------|
| Azo (Red/Yellow) | 0.100 | 1509ABH (PR48:3), 3508 (PY14) | -N=N- bond radical reaction |
| Phthalocyanine Green | 0.070 | 4506 (PG7) | Cl-substituted Cu complex |
| Quinacridone | 0.050 | 1372J (PR122), 1502 (PR176) | Polycyclic aromatic trap |
| Carbon Black | 0.046 | 8503 (37%), 7505 (45%) | Surface radical scavenging |
| Phthalocyanine Blue | 0.030 | 5504N (PB15) | Cu complex (most stable) |
| TiO₂ | 0.005 | Direct addition | Surface hydroxyl (minimal) |

**AC Foaming Interference** (per phr Color MB):

| Severity | Pigment | foamSuppression | Mechanism |
|:---------|:--------|:---------------|:----------|
| **HIGH** | Red metal-salt azo (1503 PR53:1, 1509ABH PR48:3) | 0.018–0.025 | Ba²⁺/Ca²⁺ catalyze premature AC decomposition |
| **HIGH** | Green PG7 (4506) | 0.020 | Cr³⁺ inhibits AC thermal decomposition |
| MODERATE | Carbon Black (8503, 7505) | 0.008–0.010 | Surface adsorption of AC gases |
| MILD | Yellow Azo (3508 PY14) | 0.006 | Azo decomposition byproducts |
| LOW | Blue PB15 (5504N) | 0.003 | Cu²⁺ mild, thermally stable |
| LOW | Red Quinacridone (1372J PR122) | 0.003 | No metal, low concentration |

### Causal Chain (Formulation Decision Logic)

| Priority | Factor | Nature | Determines |
|:---------|:-------|:-------|:-----------|
| 1 | Resin composition | Spec (fixed) | Temp1 range |
| 2 | Expansion (AC) | Spec (fixed) | Temp1 position (high exp → lower temp) |
| 3 | Color MB + FR | Spec (fixed) | DCP base addition + AC interference |
| 4 | Temp1 | Process (adjustable) | DCP/ZnO/Urea amount (low temp → more) |
| 5 | DCP/ZnO/Urea | Formula (adjustable) | Derived from 1-4 |
| 6 | BHT/Time/Discharge | Fine-tune | Polish quality |

### Face Diagnosis

| Face | Edges | Physical Meaning |
|------|-------|------------------|
| A | e₁ e₄ e₅ | Formula × Temp × Reaction |
| B | e₂ e₄ e₆ | Formula × Time × Reaction |
| C | e₃ e₅ e₆ | Temp × Time × Reaction |
| D | e₁ e₂ e₃ | Formula × Temp × Time |

### Self-Learning Curves

Auto-recalibrates on startup and after batch imports:
- TEMP1 per resin type (linear)
- ACDCP per resin type (quadratic)
- SUPPLY curve: supply_optimal = a×Temp1 + b

## PSO Optimizer (Factory-Safe)

| Parameter | Range | Step | Hard Limit |
|-----------|-------|------|------------|
| Temp1 | ±1.0°C | 0.2°C | 140-158°C |
| Time2 | +3/-2 min | 1 min | max 30 min |
| DCP | ±30g | 5g | CI must stay in range |
| ZnO | ±30g | 5g | — |
| BHT | ±20g | 5g | — |

## Production Form Import

- Multi-file batch with auto-calculate + save + recalibrate
- Steam pressure auto-detect (kg/cm² vs MPa)
- DosageTemp + DischargeTemp restored from CSV backup
- History limit: 3000 records

## Requirements

- Chrome/Safari/Edge, JavaScript enabled
- Chrome shows a harmless security warning when running from `file://` — does not affect functionality

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, factory precision, batch import |
| v1.1.3 | Context-aware e₅, pigment-class absorption coefficients, foamSuppression by metal content, face diagnosis UI, SVG/WebGL/manifest fixes |

## License

Proprietary — Internal use only.
