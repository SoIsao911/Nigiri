# Polymer Foaming Process Predictor v8.5 Continuation Report

## Current State
- File: index.html (~12,250 lines, ~720KB)
- Version: v8.5
- Records: 145 production records basis
- Model: 115 Score>=92 products for regression curves

## v8.5 New Feature: Improvement Recommendations Engine

### Architecture
Function: buildImprovementRecommendations(calc)
Triggers when Score < 92 or warnings exist.
New input field: Environment Temperature (envTemp, default 25C).
Winter detection: envTemp < 15C.

### Recommendation Logic

CI Low (ci < ciOptimalLow):
1. GREEN: Time2 extend to 30min
2. GREEN: Temp1 +0.5C (if < 149C)
3. GREEN: Time1 +1min (capacity impact warning)
4. YELLOW: DCP +5-30g (if ciGap > 5%)
5. YELLOW: BHT interference check

CI High (ci > ciMax):
1. GREEN: Temp1 -0.5C
2. YELLOW: DCP -5-20g

AC Decomposition Margin Low (temp2Margin < 15C):
1. GREEN: Temp2 +3C
2. YELLOW: ZnO +5-30g
3. YELLOW: Urea +5-15g

AC/DCP Ratio Deviation (> 1.5 sigma):
1. RED: AC adjust to regression target
2. RED: AC/DCP rebalance

Winter (envTemp < 15C):
1. GREEN: Time2 preventive extension to 30min

### Adjustment Ranges
Temp1: +/-0.5C max. Time1: +1min max. Time2: up to 30min.
DCP: +/-5-30g. ZnO: +5-30g. Urea: +5-15g.

## Previous Features (from v8.0b)

### Key Parameters
AC vs Expansion: 0.8253*Exp - 3.82 (RMSE=0.23)
AC/DCP Curve: 0.0112*Exp^2 - 0.02*Exp + 6.98 (RMSE=1.19)
Effective Reaction: reaction1*0.70 + total*0.30
Color MB DCP: CB 0.04-0.06, Organic 0.08-0.11
Tail DCP Loss: 2.5% per cycle. Discharge Temp: ~125-128C actual.

### DCP Cost Optimization
Trigger: Supply > 125% AND CI > ciMax. Cap: 5-20g.
Batch mode: aggregated summary only.

### Product Families
460: EVA ~7%, 28-33X, 1-2% defect
614NN/604NN: EVA ~1-5%, 20-25X, 3-4% defect
1600 special: varies, 18-20X, 5-6% defect
4000: high EVA, 30-35X, 1-2% defect
RE-137/138/139: EVA 70-85%, 17-20X, POE 11-16%

## Bug Fixes This Session
1. acDcpCurve null error - deferred to after currentCalc init
2. tailAnalysis scope - hoisted to let
3. modeSwitch ID - fixed to switchMode('lab')
4. DCP cost aggressive - ciMax target, capped 5-20g
5. Batch DCP spam - summary only
6. temp2Margin stored in currentCalc
