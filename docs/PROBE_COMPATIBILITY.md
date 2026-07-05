# Probe Compatibility - DolphinUSG

## Overview

DolphinUSG supports multiple wireless ultrasound probes manufactured by Sonostar/Sonoptek. Each probe has specific examination presets optimized for different clinical applications.

## Probe Categories

### 1. Linear Array Probes

High-frequency probes for superficial imaging.

| Probe ID | Frequency | examinations | Clinical Use |
|----------|-----------|--------------|--------------|
| PL-1C | High | LinearProbe + VesselFlow | General linear, vascular |
| SL-5H | High | LinearProbe + VesselFlow | High-resolution linear |
| UL-1H | High | LinearProbe + VesselFlow | Universal linear |
| SL-5CS | High | LinearProbe + VesselFlow | Compact linear |
| SX-1CTSB | High | LinearProbe + VesselFlow | Linear with VesselFlow |
| UX-8CB | High | LinearProbe + VesselFlow | Extended linear |

#### Default Parameters (Linear)

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| gain | 80 | 30-90 | Image gain |
| enhance | 2 | 0-4 | Edge enhancement |
| dr | 70 | 40-80 | Dynamic range |
| harmonic | 1 | 0-1 | Tissue harmonic |
| focus | 1 | 0-3 | Focus position |
| bc_WF | 3 | 0-10 | Wall filter |
| bd_angle | 60 | 0-90 | Doppler angle |
| bd_PRF_flow | 6 | 0-10 | Flow PRF |

### 2. Convex Array Probes

Lower frequency probes for deep imaging.

| Probe ID | Frequency | examinations | Clinical Use |
|----------|-----------|--------------|--------------|
| KX-1CT | Mid | PAProbe_NotLine + Linear exams | Multi-purpose |
| UX-1C | Mid | Inherits KX-1CT | Multi-purpose |
| SX-1CTA | Mid | PAProbe_NotLine | Abdominal |
| SX-8CTA | Mid | PAProbe_NotLine | Extended convex |

#### Default Parameters (Convex)

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| gain | 80 | 30-90 | Image gain |
| enhance | 2 | 0-4 | Edge enhancement |
| dr | 80 | 40-80 | Dynamic range |
| harmonic | 0 | 0-1 | Tissue harmonic |
| focus | 1 | 0-3 | Focus position |

### 3. Phased Array Probes

Cardiac-optimized probes.

| Probe ID | Frequency | examinations | Clinical Use |
|----------|-----------|--------------|--------------|
| PA-2C | Low-Mid | Cardiac + Abdomen | Cardiac imaging |

#### Default Parameters (Phased)

| Parameter | Cardiac | Abdomen |
|-----------|---------|---------|
| gain | 80 | 80 |
| zoom | 3 | 1 |
| enhance | 2 | 2 |
| dr | 80 | 80 |
| harmonic | 1 | 0 |
| focus | 2 | 2 |
| acousticalPower | 0 | 1 |
| bc_simplify | 1 | 0 |
| bc_WF | 10 | 3 |
| bc_PRF | 3 | 0 |

### 4. Specialized Probes

| Probe ID | Frequency | examinations | Clinical Use |
|----------|-----------|--------------|--------------|
| EU-1C | Very High | Eye | Ophthalmic imaging |

#### Default Parameters (Eye)

| Parameter | Default | Description |
|-----------|---------|-------------|
| gain | 80 | Image gain |
| zoom | 1 | Zoom level |
| enhance | 1 | Light enhancement |
| dr | 70 | Dynamic range |
| harmonic | 1 | Tissue harmonic |
| focus | 1 | Focus position |
| bc_gain | 75 | Color gain |
| bc_WF | 3 | Wall filter |
| bc_PRF | 0 | PRF |
| bd_PRF | 0 | Doppler PRF |

## Examination Presets

### Human Examinations (type=1)

#### Linear Probe Exams

| Exam | Key | Gain | Enhance | DR | Harmonic | Focus |
|------|-----|------|---------|-----|----------|-------|
| Thyroid | thyroid | 80 | 2 | 70 | 1 | 2 |
| Small Parts | smallParts | 80 | 2 | 70 | 1 | 1 |
| Pediatrics | pediatrics | 80 | 2 | 70 | 1 | 1 |
| Vascular | vascular | 80 | 2 | 70 | 1 | 2 |
| Carotid | carotid | 80 | 2 | 70 | 1 | 2 |
| Breast | breast | 80 | 2 | 70 | 1 | 1 |
| MSK | msk | 80 | 2 | 80 | 0 | 1 |
| Nerve | nerve | 48 | 2 | 40 | 1 | 0 |

#### Convex/Phased Probe Exams

| Exam | Key | Gain | Enhance | DR | Harmonic | Focus |
|------|-----|------|---------|-----|----------|-------|
| Abdomen | abdomen | 80 | 2 | 80 | 0 | 2 |
| Gynecology | gynecology | 80 | 2 | 80 | 0 | 1 |
| Obstetric | obstetric | 80 | 2 | 80 | 1 | 1 |
| Cardiac | cardiac | 80 | 2 | 80 | 1 | 2 |
| Urology | urology | 80 | 2 | 80 | 1 | 1 |
| Kidney | kidney | 80 | 2 | 80 | 0 | 1 |
| Lung | lung | 90 | 3 | 60 | 0 | 1 |

### Veterinary Examinations (type=2)

#### Linear Probe (Veterinary)

| Exam | Key | Gain | Enhance | DR | Harmonic | Focus |
|------|-----|------|---------|-----|----------|-------|
| Ovary | Ovary | 70 | 2 | 70 | 1 | 0 |
| Uterus | Uterus | 60 | 2 | 60 | 1 | 1 |
| Pregnancy | Pregnancy | 55 | 3 | 60 | 0 | 2 |
| Advanced Pregnancy | Advanced Pregnancy | 55 | 4 | 50 | 0 | 3 |

## Probe Inheritance

The examination configuration uses a `super` field for inheritance:

```
LinearProbe
    ├── PL-1C (VesselFlow)
    ├── SL-5H (VesselFlow)
    ├── UL-1H (VesselFlow)
    ├── SL-5CS (VesselFlow)
    ├── SX-1CTSB (VesselFlow)
    ├── UX-8CB (VesselFlow)
    └── TEST (overrides thyroid gain_N)

PAProbe_NotLine
    ├── KX-1CT
    │   └── UX-1C
    ├── SX-1CTA
    └── SX-8CTA

NotLinearProbe (base for convex)
```

## VesselFlow Mode

Available on linear probes, VesselFlow provides advanced vascular analysis:

| Parameter | Description |
|-----------|-------------|
| bd_PRF_flow | Flow PRF (6 default) |
| bc_WF | Wall filter (3 default) |
| bd_angle | Doppler angle (60 default) |

## Probe Selection Logic

```java
// Pseudo-code for probe selection
if (probe.isLinear()) {
    config = examination.json["LinearProbe"];
} else if (probe.isConvex()) {
    config = examination.json["NotLinearProbe"];
} else if (probe.isPhased()) {
    config = examination.json["PAProbe_NotLine"];
}

// Apply probe-specific overrides
if (probe.hasVesselFlow()) {
    config.put("VesselFlow", probe.vesselFlowConfig);
}

// Apply examination preset
examConfig = config[examType];
```

## Image Quality Optimization

### By Exam Type

| Exam | Priority Settings |
|------|------------------|
| Thyroid | High frequency, harmonic ON |
| Abdomen | Deep penetration, harmonic OFF |
| Cardiac | High frame rate, M-mode |
| Vascular | Color Doppler, VesselFlow |
| Pediatric | Moderate depth, high resolution |
| MSK | High resolution, compound imaging |
| Lung | Specific artifact settings |

### By Patient Size

| Patient | Adjustments |
|---------|-------------|
| Pediatric | Reduce depth, increase gain |
| Adult | Standard settings |
| Obese | Increase gain, reduce frequency |
| Elderly | Increase gain, reduce DR |
