# Plasma Refill Logic Documentation

## Overview
This document describes the automatic plasma refill simulation logic for the nHct educational tool. The refill simulates how plasma naturally refills from the interstitial space based on the patient's volemic status.

## Core Concept
Plasma refill rate depends on oncotic pressure - the more hypovolemic a patient is, the faster plasma refills from the extravascular space. This demonstrates why simply removing the "deviation amount" doesn't work for hemoconcentration.

## Refill Timing
- **Automatic Mode**: Refill occurs every 15 seconds
- **Manual Mode**: Refill occurs on button click
- **Pause Condition**: Refill is paused when user is actively dragging the slider

## Refill Amount Calculation

### Constants
```javascript
const rbcVolume = 1800; // Fixed RBC volume in mL
const normalizedHct = 38.0; // Target normalized hematocrit %
const targetTotalVolume = rbcVolume / (normalizedHct / 100); // ~4737 mL
```

### Zone-Based Refill Logic

```javascript
const currentTotalVolume = rbcVolume + plasmaVolume;
const deviation = currentTotalVolume - targetTotalVolume;
const deviationPercent = (deviation / targetTotalVolume) * 100;

// Zone thresholds
const nearTargetThreshold = 10; // ±10% of ideal

if (deviationPercent > 10) {
    // HYPERVOLEMIC: >10% above target
    refillAmount = 0;

} else if (Math.abs(deviationPercent) <= 10) {
    // NEAR TARGET: within ±10% of ideal
    refillAmount = 5; // Small constant refill (mL)

} else {
    // HYPOVOLEMIC: <-10% below target
    // Refill at 10% of current total volume
    refillAmount = Math.min(currentTotalVolume * 0.10, 300); // Cap at 300mL
}
```

### Three Zones Explained

#### 1. **HYPERVOLEMIC** (>10% above target)
- **Condition**: Patient has excess volume
- **Refill Rate**: 0 mL
- **Rationale**: No plasma refill when patient is overloaded

#### 2. **NEAR TARGET** (±10% of ideal volume)
- **Condition**: Within 10% of target volume
- **Refill Rate**: 5 mL per cycle
- **Rationale**: Minimal but continuous refill even when near or at target

#### 3. **HYPOVOLEMIC** (<-10% below target)
- **Condition**: Patient has volume deficit
- **Refill Rate**: 10% of current total volume per cycle, capped at 300 mL
- **Rationale**: Higher oncotic pressure drives more rapid plasma refill

## Example Scenarios

### Scenario A: Severely Hypovolemic
- **Current**: 3500 mL
- **Target**: 4737 mL
- **Deviation**: -1237 mL (-26.1%)
- **Refill**: 3500 × 0.10 = 350 mL → **capped at 300 mL**

### Scenario B: Mildly Hypovolemic
- **Current**: 4200 mL
- **Target**: 4737 mL
- **Deviation**: -537 mL (-11.3%)
- **Refill**: 4200 × 0.10 = 420 mL → **capped at 300 mL**

### Scenario C: Near Target
- **Current**: 4700 mL
- **Target**: 4737 mL
- **Deviation**: -37 mL (-0.8%)
- **Refill**: **5 mL** (small constant)

### Scenario D: At Target (Converged)
- **Current**: 4737 mL
- **Target**: 4737 mL
- **Deviation**: 0 mL (0%)
- **Refill**: **5 mL** (continues small refill)

### Scenario E: Hypervolemic
- **Current**: 5300 mL
- **Target**: 4737 mL
- **Deviation**: +563 mL (+11.9%)
- **Refill**: **0 mL** (no refill)

## Visual Feedback

When refill occurs, the following should update:
1. **Toast Notification**: "💧 Plasma Refill: +XXX mL"
2. **Slider Position**: Automatically adjusts to new plasma volume
3. **Test Tube**: Fluid level increases
4. **Peripheral Hct**: Value decreases (dilution effect)
5. **Volume Indicator**: Updates to show new total volume
6. **Deviation Metrics**: Update to show new excess/deficit

## User Interface Elements

### Toggle Control
```
Plasma Refill Mode:
○ Manual  ● Automatic (15s)
[Trigger Refill Now] (button - visible only in manual mode)
```

### Implementation Notes
- Default mode: **Automatic**
- Pause refill when slider `mousedown` or `touchstart` event is active
- Resume refill when slider is released
- Display refill mode status clearly
- Disable "Trigger Refill Now" button in automatic mode

## Teaching Goal

This simulation demonstrates:
1. **Why deviation-based removal fails**: Even if you remove the exact "excess" amount, plasma refills continuously
2. **Importance of nHct monitoring**: The only reliable endpoint is when Peripheral Hct converges with Normalized Hct
3. **Iterative nature of diuresis**: Clinicians must continuously monitor and adjust, not just "remove X mL and stop"

## Technical Parameters Summary

| Parameter | Value |
|-----------|-------|
| Refill Interval (Auto) | 15 seconds |
| Near Target Zone | ±10% of ideal |
| Small Refill Amount | 5 mL |
| Hypovolemic Refill Rate | 10% of current volume |
| Maximum Refill Cap | 300 mL |
| Hypervolemic Refill | 0 mL |

---

*Document created: 2025-12-11*
*Last updated: 2025-12-11*
