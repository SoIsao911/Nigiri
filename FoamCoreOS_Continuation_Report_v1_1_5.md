# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.5

Browser-based formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing. Single-file HTML — no server required.

## What's New in v1.1.5

### 1. Resin Classification (Uppercase)

| ID | Resin Mix | Rule |
|:---|:----------|:-----|
| `LDPE_ONLY` | Pure LDPE | No EVA >1kg, no POE |
| `EVA_LDPE` | EVA + LDPE | EVA >1kg direct addition |
| `POE_BLEND` | POE blend | Any POE present |

### 2. EVA_LDPE Temperature — VA% Weighted Offset

```
optTemp(EVA_LDPE) = base_curve(Exp) - weightedVA × 0.30°C
```

Example: 3000 (56% EVA, VA16%) → weighted VA ~9% → optimal -2.7°C lower.

### 3. HALS DCP Compensation

```
halsCompensation = 1.0 - min(0.25, HALS_phr × 0.10)
baseDCPNeed *= halsCompensation
```

10% reduction per 1 phr HALS, capped at 25%. Validated: 1900CUV 1.53phr HALS → Supply 80% → effective ~94%.

### 4. e₄ Dynamic Optimal — HALS Only

- Color MB → affects DCP *quantity* → handled by e₂ + e₅
- HALS → affects crosslink *efficiency* → shifts optimal ratio
- No HALS = base curve unchanged

### 5. ACDCP Curve Constraint

```
New: curve minimum ≤ 15X (prevents U-shape in operating range)
Existing: a ≥ 0 (no inverted parabola)
```

### 6. Contextual AC/DCP Warnings (B Plan)

| Scenario | Old Behavior | New Behavior |
|:---------|:------------|:-------------|
| AC/DCP low + has color MB | `[嚴重] 建議減少DCP` | `[Info] 低比值合理（配方含色母Xkg吸收DCP）` |
| AC/DCP high + has HALS | `建議增加DCP` | `高比值合理（配方含HALS降低DCP需求）` |
| AC/DCP low + no color | `[嚴重] 建議減少DCP` | Unchanged (genuine problem) |

### 7. AC/DCP Chart Redesign

- Height: 150px → 220px (professional proportion)
- Title centered with subtitle (records · resin · RMSE)
- Dual confidence bands: ±1σ (inner dark) + ±2σ (outer light)
- Footer summary bar: Current → Optimal · deviation σ
- Axis labels: AC/DCP (Y) and Expansion (X)

### Interaction Consistency Map

```
Color MB → e₂ (interference) + e₅ (baseDCPNeed)
HALS     → e₄ (dynamic optimal) + e₅ (baseDCPNeed)
FR       → e₅ (frInterference)
Each factor → specific edges only, no double-counting
```

## Quality Engine

| Edge | Meaning | Model |
|------|---------|-------|
| e₁ F─T | Temperature | Resin-specific + VA% offset (EVA_LDPE) + KNN |
| e₂ F─Ti | Color MB | Compensated (DCP supply + ZnO + CI) |
| e₃ T─Ti | Reaction | Sigmoid × Bell |
| e₄ F─R | AC/DCP | Dynamic optimal (HALS only) + KNN |
| e₅ T─R | DCP supply | HALS + color + FR compensated + KNN |
| e₆ Ti─R | CI position | Dynamic range + KNN |

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.1.3 | Context-aware e₅, pigment absorption, foamSuppression |
| v1.1.4 | e₂ compensation, AC/DCP dual-point, MC backtest, color slot export |
| v1.1.5 | Uppercase resin, EVA_LDPE VA% temp, HALS compensation, e₄ HALS-only, ACDCP constraint, contextual warnings, chart redesign |

## License

Proprietary — Internal use only.
