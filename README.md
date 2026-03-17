# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.4

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
| e₂ F─Ti | Color MB interference | Pigment-class-weighted Gaussian | ✓ |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | Context-aware f(Temp1) + KNN | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### Data-Driven Pigment Absorption (e₂)

| Pigment Class | Coefficient | MB Examples |
|:-------------|:-----------|:------------|
| Azo (Red/Yellow) | 0.100 | 1509ABH (PR48:3), 3508 (PY14) |
| Phthalocyanine Green | 0.070 | 4506 (PG7) |
| Quinacridone | 0.050 | 1372J (PR122) |
| Carbon Black | 0.046 | 8503 (37%), 7505 (45%) |
| Phthalocyanine Blue | 0.030 | 5504N (PB15) |

### AC Foaming Interference

| Severity | Pigment | foamSuppression | Mechanism |
|:---------|:--------|:---------------|:----------|
| **HIGH** | Red metal-salt azo (1503, 1509ABH) | 0.018–0.025 | Ba²⁺/Ca²⁺ catalyze AC |
| **HIGH** | Green PG7 (4506) | 0.020 | Cr³⁺ inhibits AC |
| MODERATE | Carbon Black | 0.008–0.010 | Surface adsorption |
| LOW | Blue PB15 (5504N) | 0.003 | Cu²⁺ mild |

### AC/DCP Optimal Curve

Enlarged chart (500×150px) showing:
- **Optimal curve** (auto-calibrated from history, physics-constrained)
- **Nominal ratio** (green dot): AC/DCP as formulated
- **Effective ratio** (orange circle): after deducting color MB/FR DCP absorption
- **Penalty arrow**: visual gap showing additive impact on crosslinking efficiency
- Effective point uses orange triangle (▼) — always visible even when close to nominal circle

Auto-calibration constraints:
- Quadratic coefficient a ≥ 0 (no inverted parabola)
- Expansion range ≥ 8X required
- Falls back to global curve or default if insufficient data

### Face Diagnosis

Fixed A → B → C → D display order:

| Face | Edges | Physical Meaning |
|------|-------|------------------|
| A | e₁ e₄ e₅ | Formula × Temp × Reaction |
| B | e₂ e₄ e₆ | Formula × Time × Reaction |
| C | e₃ e₅ e₆ | Temp × Time × Reaction |
| D | e₁ e₂ e₃ | Formula × Temp × Time |

## PSO Optimizer (Factory-Safe)

| Parameter | Range | Step | Hard Limit |
|-----------|-------|------|------------|
| Temp1 | ±1.0°C | 0.2°C | 140-158°C |
| Time2 | +3/-2 min | 1 min | max 30 min |
| DCP | ±30g | 5g | CI in range |
| ZnO | ±30g | 5g | — |
| BHT | ±20g | 5g | — |

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, batch import |
| v1.1.3 | Context-aware e₅, pigment-class absorption, foamSuppression by metal |
| v1.1.4 | Face order A→D, AC/DCP curve redesign (effective ratio + overlap fix), ACDCP recalibration physics constraint |

## License

Proprietary — Internal use only.
