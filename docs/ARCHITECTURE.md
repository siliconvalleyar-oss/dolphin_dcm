# Architecture - DolphinUSG

## System Overview

DolphinUSG follows a standard Android application architecture with native image processing via RenderScript and JNI, plus a DICOM networking layer based on dcm4che3.

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                       │
├─────────────────────────────────────────────────────────────┤
│  MainActivity  │ PatientActivity │ ReportActivity │ Settings│
│  (Ultrasound)  │   (Patient)     │   (Reports)    │ (Config)│
└────────┬───────────────┬───────────────┬───────────┬────────┘
         │               │               │           │
┌────────▼───────────────▼───────────────▼───────────▼────────┐
│                     VIEW LAYER                              │
├─────────────────────────────────────────────────────────────┤
│  USBMImageView │ USColorView │ USMotionView │ USPWLineView  │
│  USBiopsyView  │ USScaleView │ USMarkView   │ LoopView      │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   BUSINESS LAYER                            │
├─────────────────────────────────────────────────────────────┤
│  Image Processing │ Measurement Kit │ Exam Config │ DICOM   │
│  (RenderScript)   │ (measurekit)    │ (JSON)      │ Service │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    NATIVE LAYER                             │
├─────────────────────────────────────────────────────────────┤
│  libvessel_flow.so │ librs.*.so │ RenderScript Runtime      │
│  (Vessel Analysis) │ (Enhance)  │ (GPU Compute)             │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   HARDWARE LAYER                            │
├─────────────────────────────────────────────────────────────┤
│  WiFi Probe Connection │ USB Connection │ DICOM PACS        │
│  (TCP/UDP Stream)      │ (USB Host)     │ (dcm4che3)        │
└─────────────────────────────────────────────────────────────┘
```

## Component Details

### 1. Activities

#### MainActivity (Core Scanning)
- **Purpose**: Main ultrasound scanning interface
- **Inner Classes**: ~30 anonymous/inner classes (a0 through z0)
- **Responsibilities**:
  - Real-time image streaming from probe
  - Mode switching (B/M/Color/PW)
  - Gain/Depth/Focus control
  - Freeze/Unfreeze
  - Image capture
  - Measurement activation

#### PatientActivity
- **Purpose**: Patient data entry and management
- **Fields**: Name, ID, BirthDate, Sex, Referring Physician
- **Integration**: DICOM Worklist (MWL)

#### ReportActivity
- **Purpose**: Generate and share ultrasound reports
- **Implementation**: WebView-based with JavaScript bridge
- **Flow**: Image → HTML Template → Canvas Capture → Share

#### SettingActivity
- **Purpose**: Application configuration
- **Settings**: WiFi probe connection, DICOM server, display preferences

### 2. Custom Views

| View Class | Rendering Mode | Description |
|-----------|---------------|-------------|
| `USBMImageView` | B-Mode | Grayscale ultrasound image |
| `USColorView` | Color Flow | Color Doppler overlay |
| `USMotionView` | M-Mode | Time-motion display |
| `USPWLineView` | PW Line | Sample volume indicator |
| `USPWExLineView` | PW Extended | Extended sample volume |
| `USPWMotionView` | PW M-Mode | Spectral Doppler display |
| `USPWExMotionView` | PW Ext M-Mode | Extended spectral display |
| `USBiopsyView` | Overlay | Needle guidance |
| `USScaleView` | Overlay | Measurement scale |
| `USMarkView` | Overlay | Annotation markers |

### 3. Measurement Kit (com.sonoptek.measurekit)

27 measurement classes providing:

| Tool | Class Range | Function |
|------|------------|----------|
| Distance | a-e | Linear measurement |
| Area | f-j | Region area calculation |
| Volume | k-m | 3D volume estimation |
| Angle | n-p | Angular measurement |
| Trace | q-t | Freehand trace/area |
| Doppler | u-z | Velocity/flow measurement |
| Heart Rate | a0-d0 | Cardiac calculations |

### 4. Image Processing Pipeline

```
Raw Probe Data
      │
      ▼
┌─────────────┐
│  DSC (Digital│ ← librs.dscenhance.so
│  Scan Conv.) │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Image      │ ← librs.imageenhance.so
│  Enhancement│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Extended   │ ← librs.imageexenhancer*.so
│  Enhancer   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Vessel     │ ← libvessel_flow.so
│  Analysis   │
└──────┬──────┘
       │
       ▼
   Display
```

### 5. DICOM Layer (dcm4che3)

```
┌─────────────────────────────────────┐
│         DICOM Association          │
├─────────────────────────────────────┤
│  Association  │ Presentation  │     │
│  Negotiation  │ Context       │     │
└───────┬───────┴───────┬───────┘     │
        │               │             │
┌───────▼───────┐ ┌─────▼──────┐     │
│  SCU Services │ │ SCP Services│     │
│  (Client)     │ │ (Server)    │     │
├───────────────┤ ├────────────┤     │
│ C-FIND        │ │ C-ECHO     │     │
│ C-STORE       │ │ C-FIND     │     │
│ C-MOVE        │ │ C-STORE    │     │
│ C-GET         │ │ C-MOVE     │     │
│               │ │ MPPS       │     │
└───────────────┘ └────────────┘     │
```

## Data Flow

### 1. Real-time Imaging
```
Probe → WiFi/USB → Frame Buffer → RenderScript → Custom View → Screen
```

### 2. DICOM Export
```
Freeze Frame → DICOM Encode → C-STORE → PACS Server
```

### 3. Report Generation
```
Image + Observations → WebView (html2canvas) → Base64 PNG → Share
```

## Threading Model

| Thread | Purpose |
|--------|---------|
| Main/UI | Android UI updates |
| Probe Stream | Continuous frame reception |
| DICOM Service | Network DICOM operations |
| Image Processing | RenderScript GPU compute |
| File I/O | DICOM file read/write |

## Obfuscated Packages

| Original Package | Likely Purpose |
|-----------------|---------------|
| `a/a/l/a` | Protocol handler |
| `a/d/a/j` | DICOM service implementation |
| `a/f/f` | Network communication |
| `a/f/i` | Image processing |
| `a/l/a/a` | Data serialization |
| `b/a/a` | DICOM library wrapper |
| `c/a/a/a` | Secondary subsystem |
