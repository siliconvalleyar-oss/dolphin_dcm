# DolphinUSG - Wireless Ultrasound DICOM

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/Android-6.0%2B-green.svg)](https://developer.android.com/about/versions/marshmallow)
[![DICOM](https://img.shields.io/badge/DICOM-dcm4che3-orange.svg)](http://www.dcm4che.org)

> **Reverse-engineered analysis of the DolphinUSG wireless ultrasound application**

## Overview

DolphinUSG is a decompiled Android APK for a **wireless medical ultrasound imaging application** developed by **Sonostar/Sonoptek**. The app connects to wireless ultrasound probes via WiFi/USB and provides real-time B-mode, M-mode, Color Doppler, PW Doppler imaging with DICOM support for PACS integration.

**Version**: 1.1.1  
**Package**: `com.sonoptek.wirelessusg3`  
**Developer**: Sonostar (com.sonostar) / Sonoptek (com.sonoptek)

## Quick Start

```bash
# Clone the repository
git clone https://github.com/siliconvalleyar-oss/dolphin_dcm.git

# Navigate to project
cd dolphin_dcm

# View documentation
cat docs/ARCHITECTURE.md
```

## Project Structure

```
dolphin_dcm/
├── README.md                    # This file
├── .gitignore                   # Git ignore rules
├── docs/                        # Documentation
│   ├── ARCHITECTURE.md          # System architecture
│   ├── DICOM_INTEGRATION.md     # DICOM/PACS integration guide
│   ├── PROBE_COMPATIBILITY.md   # Supported probes
│   ├── IMAGING_MODES.md         # Imaging modes reference
│   ├── NETWORK_PROTOCOL.md      # Network/connection protocol
│   ├── SECURITY.md              # Security analysis
│   └── ANALYSIS.md              # Full reverse engineering analysis
├── src/                         # Source code analysis
│   └── classes_analysis.md      # DEX class analysis
├── assets/                      # App assets
│   ├── examination.json         # Human exam presets
│   ├── examination_vetus.json   # Veterinary exam presets
│   ├── report.html              # English report template
│   ├── report_cn.html           # Chinese report template
│   └── jsonComm.js              # WebView JS bridge
└── lib/                         # Native libraries
    ├── arm64-v8a/               # 64-bit ARM
    ├── armeabi-v7a/             # 32-bit ARM
    ├── x86_64/                  # 64-bit x86
    └── x86/                     # 32-bit x86
```

## Key Features

| Feature | Description |
|---------|-------------|
| **B-Mode** | Standard 2D grayscale imaging |
| **M-Mode** | Time-motion display for cardiac |
| **Color Doppler** | Blood flow visualization |
| **PW Doppler** | Spectral Doppler analysis |
| **Biopsy Guide** | Needle guidance overlay |
| **DICOM** | PACS integration (C-STORE, C-FIND, C-MOVE) |
| **MWL** | Modality Worklist support |
| **Reports** | HTML-based report generation |
| **VesselFlow** | Native vessel analysis (libvessel_flow.so) |

## Supported Probes

### Linear Probes
- PL-1C, SL-5H, UL-1H, SL-5CS, SX-1CTSB, UX-8CB

### Convex/Phased Probes
- KX-1CT, UX-1C, SX-1CTA, SX-8CTA, PA-2C

### Specialized
- EU-1C (ophthalmic/eye)

## Documentation

- [Architecture](docs/ARCHITECTURE.md) - System design and components
- [DICOM Integration](docs/DICOM_INTEGRATION.md) - PACS connectivity
- [Probe Compatibility](docs/PROBE_COMPATIBILITY.md) - Supported hardware
- [Imaging Modes](docs/IMAGING_MODES.md) - Mode reference
- [Network Protocol](docs/NETWORK_PROTOCOL.md) - Connection details
- [Security Analysis](docs/SECURITY.md) - Vulnerability assessment
- [Full Analysis](docs/ANALYSIS.md) - Complete reverse engineering report

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Language | Java (obfuscated) |
| UI Framework | Android SDK + AndroidX |
| Image Processing | RenderScript (GPU) |
| DICOM Library | dcm4che3 |
| Report Rendering | WebView + html2canvas |
| Native Code | C/C++ via JNI |
| JS Bridge | jsonComm.js |

## License

This project is for **educational and research purposes only**. The original application and its intellectual property belong to Sonostar/Sonoptek.

## Disclaimer

This analysis is performed for educational purposes to understand medical device software architecture. Always follow proper regulatory guidelines when working with medical devices.
