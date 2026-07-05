# Full Reverse Engineering Analysis - DolphinUSG

## Executive Summary

This document provides a comprehensive reverse engineering analysis of the DolphinUSG wireless ultrasound Android application (v1.1.1). The analysis covers application architecture, code structure, security posture, and technical capabilities.

**Application**: DolphinUSG  
**Version**: 1.1.1  
**Package**: `com.sonoptek.wirelessusg3`  
**Developer**: Sonostar/Sonoptek  
**Platform**: Android  
**Type**: Native Android (not Flutter)

## 1. APK Structure

### File Inventory

| File | Size | Description |
|------|------|-------------|
| `classes.dex` | 2,893,872 bytes | DEX bytecode |
| `libvessel_flow.so` | 494,004 bytes (arm64) | Vessel analysis native |
| `libRSSupport.so` | 1,236,904 bytes (arm64) | RenderScript runtime |
| `librs.imageenhance.so` | 133,416 bytes (arm64) | Image enhancement |
| `librs.dscenhance.so` | 132,888 bytes (arm64) | DSC enhancement |
| `examination.json` | 462 lines | Exam presets |
| `examination_vetus.json` | 44 lines | Vet exam presets |
| `report.html` | 101 lines | English report |
| `report_cn.html` | 101 lines | Chinese report |
| `jsonComm.js` | 37 lines | JS bridge |

### DEX Statistics

| Metric | Value |
|--------|-------|
| Magic | `dex\n035\0` |
| File Size | 2,893,872 bytes |
| Header Size | 112 bytes |
| String IDs | 19,829 |
| Type IDs | 2,646 |
| Proto IDs | 4,335 |
| Method IDs | 13,875 |

## 2. Package Analysis

### Main Application Package

```
com/sonostar/wirelessusg/
├── MainActivity          # Main scanning activity
│   ├── $a through $z0    # ~30 inner classes
├── PatientActivity       # Patient management
├── ReportActivity        # Report generation
├── SettingActivity       # Configuration
├── MyApplication         # Application class
├── LoopView              # Scroll selector
├── USButton              # Custom button
├── a through u           # Helper classes
└── usview/               # Custom views
    ├── USBMImageView     # B-Mode view
    ├── USColorView       # Color view
    ├── USMotionView      # M-Mode view
    ├── USPWLineView      # PW line
    ├── USPWExLineView    # Extended PW
    ├── USPWMotionView    # PW M-mode
    ├── USBiopsyView      # Biopsy guide
    ├── USScaleView       # Scale overlay
    └── [a-k]             # View helpers
```

### Measurement Kit Package

```
com/sonoptek/measurekit/
├── USMarkView            # Measurement marker
└── [a0-d0]               # 27 measurement tools
```

### JNI Package

```
com/sonoptek/jni/
└── VesselFlow            # Vessel analysis bridge
```

### Obfuscated Packages

| Package | Likely Purpose | Class Count |
|---------|---------------|-------------|
| `a/a/l/a` | Protocol handler | ~15 |
| `a/d/a/j` | DICOM service | ~20 |
| `a/f/f` | Network comm | ~15 |
| `a/f/i` | Image processing | ~25 |
| `a/l/a/a` | Serialization | ~15 |
| `b/a/a` | DICOM library | ~200 |
| `c/a/a/a` | Secondary sys | ~17 |

## 3. Class Analysis

### Application Entry Points

| Class | Inner Classes | Methods | Description |
|-------|---------------|---------|-------------|
| `MainActivity` | 30 | 50+ | Core scanning |
| `PatientActivity` | 0 | 10+ | Patient data |
| `ReportActivity` | 3 | 15+ | Reports |
| `SettingActivity` | 2 | 10+ | Settings |
| `MyApplication` | 1 | 5+ | App state |

### Key Method Signatures Found

```java
// Connection callbacks
onConnectionAccepted()
onConnectionEstablished()
onConnectionFailed()
onConnectionRejected()
onConnectionRejectedBlacklisted()

// UI operations
freeze()
loadImage()
saveReport()
measure()
measure_volume()

// DICOM operations
openFile()
openFileDescriptor()
openFileOutput()
```

## 4. Native Library Analysis

### Vessel Flow Library (libvessel_flow.so)

| Architecture | Size | Purpose |
|--------------|------|---------|
| arm64-v8a | 494 KB | Vessel analysis |
| armeabi-v7a | 494 KB | Vessel analysis |
| x86_64 | 890 KB | Vessel analysis |
| x86 | 844 KB | Vessel analysis |

### RenderScript Libraries

| Library | Size (arm64) | Purpose |
|---------|--------------|---------|
| `librs.imageenhance.so` | 133 KB | Image enhancement |
| `librs.imageexenhancer.so` | 133 KB | Extended enhancement |
| `librs.imageexenhancer_full.so` | 198 KB | Full pipeline |
| `librs.imageexenhancer_v1.so` | 198 KB | Enhanced v1 |
| `librs.dscenhance.so` | 133 KB | DSC enhancement |
| `libRSSupport.so` | 1.2 MB | Runtime support |
| `librsjni.so` | 72 KB | JNI bridge |
| `librsjni_androidx.so` | 70 KB | AndroidX JNI |

## 5. DICOM Implementation

### dcm4che3 Library Components

| Package | Classes | Purpose |
|---------|---------|---------|
| `org.dcm4che3.data` | 10+ | Data structures |
| `org.dcm4che3.image` | 10+ | Image processing |
| `org.dcm4che3.imageio` | 20+ | ImageIO codecs |
| `org.dcm4che3.io` | 5+ | Stream I/O |
| `org.dcm4che3.net` | 15+ | Networking |
| `org.dcm4che3.net.service` | 20+ | SCP services |
| `org.dcm4che3.tool` | 10+ | CLI tools |
| `org.dcm4che3.media` | 3+ | DICOMDIR |

### DICOM Services Implemented

| Service | Class | Role |
|---------|-------|------|
| C-ECHO | `BasicCEchoSCP` | SCP |
| C-FIND | `BasicCFindSCP` | SCP |
| C-STORE | `BasicCStoreSCP` | SCP |
| C-MOVE | `BasicCMoveSCP` | SCP |
| C-GET | `BasicCGetSCP` | SCP |
| MPPS | `BasicMPPSSCP` | SCP |

## 6. Image Processing Pipeline

### RenderScript Pipeline

```
Input Frame (RF Data)
    │
    ▼
┌─────────────────┐
│ Envelope Detection│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Log Compression  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ DSC Conversion   │ ← librs.dscenhance.so
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Image Enhancement│ ← librs.imageenhance.so
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Extended Process │ ← librs.imageexenhancer*.so
└────────┬────────┘
         │
         ▼
    Display Buffer
```

### Vessel Flow Processing

```
Color Flow Data
    │
    ▼
┌─────────────────┐
│ Wall Filter      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ FFT Processing   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Velocity Calc    │ ← libvessel_flow.so
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Color Mapping    │
└────────┬────────┘
         │
         ▼
    Color Overlay
```

## 7. WebView Bridge

### Communication Pattern

```
┌─────────────────────────────────────────────────┐
│  Native (Java)          WebView (JavaScript)    │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────────┐      ┌──────────────┐         │
│  │ Call JS       │─────►│ loadJSON     │         │
│  │ Function      │      │ CallFunc     │         │
│  └──────────────┘      └──────┬───────┘         │
│                               │                  │
│                               ▼                  │
│                        ┌──────────────┐         │
│                        │ eval(func)   │         │
│                        └──────┬───────┘         │
│                               │                  │
│                               ▼                  │
│                        ┌──────────────┐         │
│                        │ Execute      │         │
│                        │ Function     │         │
│                        └──────┬───────┘         │
│                               │                  │
│                               ▼                  │
│  ┌──────────────┐      ┌──────────────┐         │
│  │ loadJSFor    │◄─────│ makeJSON     │         │
│  │ MTWithJson   │      │ With         │         │
│  └──────────────┘      └──────────────┘         │
│                                                  │
└─────────────────────────────────────────────────┘
```

### JS Bridge Protocol

```javascript
// Native → JS
window.hello.loadJSForMTWithJson(JSON.stringify({
    Function: "functionName",
    Param: "parameter"
}));

// JS → Native
jsonComm.makeJSONWith("functionName", ["param"]);
// Returns: '{"Function":"functionName","Param":"param"}'
```

## 8. Configuration System

### Examination Presets (examination.json)

**Structure**:
```json
{
    "LinearProbe": {
        "thyroid": { "gain": "80", "enhance": "2", ... },
        "smallParts": { ... }
    },
    "NotLinearProbe": {
        "abdomen": { ... }
    },
    "KX-1CT": {
        "super": "PAProbe_NotLine",
        ...overrides...
    }
}
```

**Inheritance Model**:
- `LinearProbe` → base linear config
- `NotLinearProbe` → base convex config
- `PAProbe_NotLine` → inherits NotLinearProbe
- Probe-specific → inherits from parent

### Veterinary Config (examination_vetus.json)

```json
{
    "LinearProbe": {
        "Ovary": { "type": "2", ... },
        "Uterus": { ... },
        "Pregnancy": { ... },
        "Advanced Pregnancy": { ... }
    }
}
```

## 9. Security Findings

### Critical Issues

1. **No TLS for DICOM** - All network communication is plaintext
2. **No certificate validation** - MITM attacks possible
3. **eval() in JS bridge** - Code injection risk
4. **Expired signing certificate** - Cannot update

### High Issues

1. **No probe authentication** - Unauthorized access possible
2. **Patient data unencrypted** - HIPAA violation
3. **Hardcoded network config** - Security through obscurity

### Medium Issues

1. **Outdated dependencies** - jQuery 1.8.3, html2canvas 0.5.0-beta3
2. **Heavy obfuscation** - Difficult security audit
3. **No input validation** - WebView vulnerabilities

## 10. Recommendations

### Immediate

1. Implement TLS for all DICOM communication
2. Replace eval() with safe function mapping
3. Renew signing certificate
4. Add certificate validation

### Short-term

1. Implement probe authentication
2. Encrypt patient data at rest
3. Add certificate pinning
4. Update dependencies

### Long-term

1. Security audit (penetration test)
2. Compliance certification (HIPAA, GDPR)
3. Implement security monitoring
4. Regular security updates

## Conclusion

DolphinUSG is a feature-rich wireless ultrasound application with comprehensive DICOM integration. However, significant security vulnerabilities exist, particularly in network communication and data protection. Immediate action is required to address critical security issues, especially for medical device compliance.

---

**Analysis Date**: 2026-07-05  
**Analyst**: MiMoCode  
**Classification**: Confidential - Educational Purpose Only
