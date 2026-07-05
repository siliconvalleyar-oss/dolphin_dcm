# Imaging Modes - DolphinUSG

## Overview

DolphinUSG supports multiple ultrasound imaging modes for different clinical applications. Each mode provides specific information about tissue structure, motion, or blood flow.

## Mode Summary

| Mode | Full Name | Primary Use | Image Type |
|------|-----------|-------------|------------|
| B | Brightness | Anatomical structure | 2D grayscale |
| M | Motion | Cardiac wall motion | Time-motion display |
| Color | Color Doppler | Blood flow direction | Color overlay |
| PW | Pulsed Wave | Blood velocity | Spectral Doppler |
| PW Ex | Extended PW | Extended velocity | Spectral + M-mode |
| Biopsy | Needle Guide | Interventional | Overlay guide |

## 1. B-Mode (Brightness Mode)

### Description
Standard 2D grayscale imaging showing anatomical structures in real-time.

### Parameters

| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| gain | 30-90 | 80 | Overall image brightness |
| dr | 40-80 | 70-80 | Dynamic range (contrast) |
| enhance | 0-4 | 2 | Edge enhancement |
| harmonic | 0-1 | varies | Tissue harmonic imaging |
| focus | 0-3 | varies | Focus position |
| depth | variable | varies | Imaging depth |
| zoom | 0-3 | 0 | Image magnification |
| compound | 0-1 | varies | Spatial compounding |

### Image Processing Pipeline

```
Raw RF Data
    │
    ▼
┌─────────────┐
│ Envelope     │  Extract amplitude
│ Detection    │  from RF signal
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Log          │  Compress dynamic
│ Compression  │  range
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ DSC          │  Digital Scan
│ (Scan Conv)  │  Conversion
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Edge         │  Spatial filtering
│ Enhancement  │  for detail
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Frame        │  Temporal
│ Averaging    │  smoothing
└──────┬──────┘
       │
       ▼
   Display
```

### Tissue Harmonic Imaging (THI)

When `harmonic=1`:
- Transmits at fundamental frequency
- Receives at 2x frequency (harmonic)
- Reduces artifacts
- Improves contrast resolution
- Best for: Abdomen, cardiac, deep structures

## 2. M-Mode (Motion Mode)

### Description
Displays tissue motion over time along a single scan line. Primarily used for cardiac imaging.

### Parameters

| Parameter | Description |
|-----------|-------------|
| Sweep Speed | Time display rate |
| Depth | Displayed depth range |
| Gain | Same as B-mode |

### Display Layout

```
┌───────────────────────────────────────┐
│           B-Mode Image                │
│                                       │
│     ┌─── M-mode sample line           │
│     │                                 │
│     ▼                                 │
│    ╱│╲                                │
│                                       │
└───────────────────────────────────────┘
┌───────────────────────────────────────┐
│         M-Mode Display                │
│  ─────╱╲╱╲╱╲──────  Time →           │
│       ╲╱╲╱╲╱                        │
│  ─────╱╲╱╲╱╲──────                   │
└───────────────────────────────────────┘
```

### Clinical Applications

| Application | Settings |
|-------------|----------|
| LV Function | High sweep speed |
| Valve Motion | Moderate sweep speed |
| Wall Thickness | Low sweep speed |
| Fetal Heart | Moderate sweep, high frame |

## 3. Color Doppler (Color Flow)

### Description
Overlays color-coded blood flow information on B-mode image.

### Parameters

| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| bc_gain | 0-100 | 75 | Color gain |
| bc_WF | 0-10 | 3 | Wall filter |
| bc_PRF | 0-10 | varies | Pulse repetition frequency |
| bd_angle | 0-90 | 60 | Doppler angle correction |
| bc_simplify | 0-1 | varies | Color simplification |
| bc_darkPassFilter | 0-1 | varies | Dark pass filter |
| ROI position | variable | - | Region of interest |
| ROI size | variable | - | Sample volume size |

### Color Map

```
        RED: Flow toward probe
           ╱╲
          ╱  ╲
         ╱    ╲
BLACK ──╱──────╲── BLACK (no flow)
         ╲    ╱
          ╲  ╱
           ╲╱
        BLUE: Flow away from probe
```

### Wall Filter (bc_WF)

| Value | Effect |
|-------|--------|
| 0 | Off (all frequencies passed) |
| 1 | Low filter |
| 3 | Medium filter (default) |
| 5 | High filter |
| 10 | Very high filter |

### Color Simplify (bc_simplify)

When enabled:
- Reduces color variations
- Improves flow visualization
- Useful for low-flow states

## 4. PW Doppler (Pulsed Wave)

### Description
Measures blood velocity at a specific sample volume location.

### Display Layout

```
┌───────────────────────────────────────┐
│           B-Mode Image                │
│                                       │
│     ──┼── PW sample volume            │
│       │   (gate)                      │
│                                       │
└───────────────────────────────────────┘
┌───────────────────────────────────────┐
│         Spectral Display              │
│        ╱╲    ╱╲    ╱╲                │
│  ──────╱──╲╱──╲╱──╲───── 0 line      │
│       ╲  ╱╲  ╱╲  ╱                   │
│        ╲╱  ╲╱  ╲╱                    │
│            Time →                     │
└───────────────────────────────────────┘
```

### Parameters

| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| bd_PRF | 0-10 | varies | Pulse repetition frequency |
| sampleVolume | variable | - | Gate size |
| angle | 0-90 | 60 | Angle correction |
| baseline | variable | - | Spectral baseline |
| invert | 0-1 | 0 | Spectral inversion |
| scale | variable | - | Velocity scale |

### PRF (Pulse Repetition Frequency)

| PRF | Depth Range | Velocity Range |
|-----|-------------|----------------|
| Low | Deep | Low velocities |
| High | Shallow | High velocities |

## 5. Extended PW (PW Ex)

### Description
Combines PW spectral display with M-mode for simultaneous motion and velocity assessment.

### Display Layout

```
┌───────────────────────────────────────┐
│           B-Mode Image                │
│                                       │
│     ──┼── PW sample volume            │
│       │                               │
└───────────────────────────────────────┘
┌───────────────────────────────────────┐
│      Combined PW + M-Mode             │
│  ───╱╲╱╲───  Spectral                 │
│     ╲╱╲╱                             │
│  ──────────  M-mode trace             │
│  ╱╲╱╲╱╲╱╲                            │
│  ╲╱╲╱╲╱╲╱                            │
│            Time →                     │
└───────────────────────────────────────┘
```

## 6. Biopsy Guide

### Description
Needle guidance overlay for interventional procedures.

### Features

| Feature | Description |
|---------|-------------|
| Guide Line | Virtual needle path |
| Depth Marks | Depth indicators |
| Angle Adjustment | Configurable guide angle |
| Safe Zone | Target area highlighting |

### Guide Types

| Type | Angle | Use |
|------|-------|-----|
| 0° | Straight | Parallel approach |
| 15° | 15° | Angled approach |
| 30° | 30° | Standard biopsy |
| 45° | 45° | Steep approach |

## Mode Switching

### Priority Order

```
B-Mode (always active)
    │
    ├── + M-Mode
    │
    ├── + Color Doppler
    │
    ├── + PW Doppler
    │
    ├── + Extended PW
    │
    └── + Biopsy Guide
```

### State Transitions

| From | To | Trigger |
|------|-----|---------|
| B | B+M | M-mode button |
| B | B+Color | Color button |
| B | B+PW | PW button |
| B+Color | B+PW | PW button (replaces Color) |
| Any | B | Freeze/Mode reset |

## Performance Metrics

| Mode | Frame Rate | Resolution | Penetration |
|------|------------|------------|-------------|
| B | 30-60 fps | High | Good |
| M | 30-60 fps | High | Good |
| Color | 15-30 fps | Medium | Medium |
| PW | 15-30 fps | High | Medium |
| PW Ex | 15-30 fps | High | Medium |

## Optimization Tips

### For Better B-Mode
- Increase gain for hypoechoic structures
- Decrease gain for hyperechoic structures
- Enable harmonic for deep structures
- Use compound imaging for speckle reduction

### For Better Color
- Adjust PRF to match flow velocity
- Increase color gain until noise appears, then reduce
- Use wall filter to remove vessel wall motion
- Optimize ROI size for frame rate

### For Better PW
- Align sample volume with flow direction
- Use appropriate PRF for expected velocities
- Adjust baseline to center spectrum
- Enable angle correction for velocity accuracy
