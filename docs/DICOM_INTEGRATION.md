# DICOM Integration - DolphinUSG

## Overview

DolphinUSG integrates with hospital PACS (Picture Archiving and Communication System) using the DICOM standard via the embedded **dcm4che3** library. This enables the ultrasound modality to store, query, and retrieve medical images.

## DICOM Services

### Service Classes Implemented

| Service | Role | Class | Description |
|---------|------|-------|-------------|
| C-ECHO | SCP | `BasicCEchoSCP` | Connection verification (ping) |
| C-FIND | SCP | `BasicCFindSCP` | Query patient/study info |
| C-STORE | SCP | `BasicCStoreSCP` | Receive images from PACS |
| C-MOVE | SCP | `BasicCMoveSCP` | Move images to another AE |
| C-GET | SCP | `BasicCGetSCP` | Retrieve images |
| MPPS | SCP | `BasicMPPSSCP` | Modality Performed Procedure Step |

### AE (Application Entity) Configuration

```
┌─────────────────────────────────────────────┐
│  LOCAL MODALITY (This App)                  │
├─────────────────────────────────────────────┤
│  AE Title:    DCM1_THE_AE_TITLE             │
│  Port:        (configured in settings)      │
│  Role:        SCU + SCP                     │
└─────────────────────────────────────────────┘
                    │
                    │ DICOM Association
                    ▼
┌─────────────────────────────────────────────┐
│  PACS SERVER                                │
├─────────────────────────────────────────────┤
│  AE Title:    DCM2_THE_AE_TITLE             │
│  Port:        DCM2_PORT                     │
│  Role:        SCP                           │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│  MWL SERVER (Modality Worklist)             │
├─────────────────────────────────────────────┤
│  AE Title:    MWL_THE_AE_TITLE              │
│  Port:        MWL_PORT                      │
│  Role:        SCP                           │
└─────────────────────────────────────────────┘
```

## DICOM Data Elements

### Patient Module

| Tag | Element | Description |
|-----|---------|-------------|
| (0010,0010) | PatientName | Patient's full name |
| (0010,0020) | PatientID | Unique patient identifier |
| (0010,0030) | PatientBirthDate | Date of birth (YYYYMMDD) |
| (0010,0040) | PatientSex | M/F/O |
| (0010,1000) | OtherPatientIDs | Additional IDs |
| (0010,1001) | OtherPatientNames | Additional names |
| (0010,0021) | IssuerOfPatientID | ID issuer |

### Study Module

| Tag | Element | Description |
|-----|---------|-------------|
| (0008,0020) | StudyDate | Examination date |
| (0008,0030) | StudyTime | Examination time |
| (0008,1030) | StudyDescription | Study description |
| (0008,0050) | AccessionNumber | Study accession |
| (0008,0090) | ReferringPhysicianName | Ordering physician |

### Series Module

| Tag | Element | Description |
|-----|---------|-------------|
| (0008,0060) | Modality | US (Ultrasound) |
| (0008,103E) | SeriesDescription | Series description |
| (0008,0070) | Manufacturer | Sonostar/Sonoptek |
| (0008,1090) | ManufacturerModelName | DolphinUSG |

### Equipment Module

| Tag | Element | Description |
|-----|---------|-------------|
| (0008,0070) | Manufacturer | Device manufacturer |
| (0008,1090) | ManufacturerModelName | Model name |
| (0018,1020) | SoftwareVersions | App version (1.1.1) |

## Transfer Syntaxes

### Explicit VR Little Endian (Default)
```
1.2.840.10008.1.2.1
```
Most common transfer syntax for DICOM networking.

### JPEG Compression
| UID | Syntax |
|-----|--------|
| 1.2.840.10008.1.2.4.50 | JPEG Baseline (8-bit) |
| 1.2.840.10008.1.2.4.51 | JPEG Extended (12-bit) |
| 1.2.840.10008.1.2.4.57 | JPEG Lossless |
| 1.2.840.10008.1.2.4.70 | JPEG Lossless First-Order |

### JPEG 2000
| UID | Syntax |
|-----|--------|
| 1.2.840.10008.1.2.4.90 | JPEG 2000 Lossless |
| 1.2.840.10008.1.2.4.91 | JPEG 2000 Lossy |

### JPEG-LS
| UID | Syntax |
|-----|--------|
| 1.2.840.10008.1.2.4.80 | JPEG-LS Lossless |
| 1.2.840.10008.1.2.4.81 | JPEG-LS Lossy |

### MPEG
| UID | Syntax |
|-----|--------|
| 1.2.840.10008.1.2.4.100 | MPEG2 HD |
| 1.2.840.10008.1.2.4.101 | MPEG-4 HD |

## Modality Worklist (MWL)

### Query Parameters

```java
// MWL Query Keys
PatientName:       "DOE^JOHN"
PatientID:         "12345"
StudyDate:         "20240101-20241231"
Modality:          "US"
AccessionNumber:   ""
ScheduledStationAET: "MWL_AE_TITLE"
```

### MWL Response Fields

| Field | DICOM Tag | Description |
|-------|-----------|-------------|
| Patient Name | (0010,0010) | Patient name |
| Patient ID | (0010,0020) | Patient ID |
| Birth Date | (0010,0030) | Date of birth |
| Sex | (0010,0040) | Gender |
| Study Date | (0008,0020) | Scheduled date |
| Study Time | (0008,0030) | Scheduled time |
| Accession # | (0008,0050) | Accession number |
| Referring MD | (0008,0090) | Referring physician |
| Study Desc | (0008,1030) | Study description |
| Modality | (0008,0060) | US |

## Connection Protocol

### Association Negotiation

```
1. A-ASSOCIATE-RQ (Request)
   ├── Called AET: DCM2_THE_AE_TITLE
   ├── Calling AET: DCM1_THE_AE_TITLE
   └── Presentation Contexts:
       ├── 1.2.840.10008.5.1.4.1.1.2 (CT Storage)
       └── 1.2.840.10008.1.3.10 (DICOMDIR)

2. A-ASSOCIATE-AC (Accept)
   ├── Result: Accepted
   └── Transfer Syntax: 1.2.840.10008.1.2.1

3. DIMSE Commands
   ├── C-STORE-RQ → C-STORE-RSP
   ├── C-FIND-RQ → C-FIND-RSP (pending) → C-FIND-RSP (complete)
   └── C-MOVE-RQ → C-MOVE-RSP (pending) → C-MOVE-RSP (success)

4. A-RELEASE-RQ → A-RELEASE-RP
```

### Connection Lifecycle Callbacks

```java
onConnectionAccepted()      // Association accepted
onConnectionEstablished()   // Connection ready
onConnectionFailed()        // Connection error
onConnectionRejected()      // Association rejected
onConnectionRejectedBlacklisted()  // Blacklisted peer
```

## DICOM File Structure

```
┌─────────────────────────────┐
│  Preamble (128 bytes)       │
├─────────────────────────────┤
│  Magic: "DICM"             │
├─────────────────────────────┤
│  Meta Information Group     │
│  (0002,xxxx)               │
├─────────────────────────────┤
│  Patient Module             │
│  (0010,xxxx)               │
├─────────────────────────────┤
│  Study Module               │
│  (0008,xxxx)               │
├─────────────────────────────┤
│  Series Module              │
│  (0008,xxxx)               │
├─────────────────────────────┤
│  Image Module               │
│  (0028,xxxx)               │
├─────────────────────────────┤
│  Pixel Data                 │
│  (7FE0,0010)               │
└─────────────────────────────┘
```

## dcm4che3 Library Components

### Core
| Package | Purpose |
|---------|---------|
| `org.dcm4che3.data` | DICOM data structures |
| `org.dcm4che3.io` | DICOM stream I/O |
| `org.dcm4che3.net` | DICOM networking |

### Image
| Package | Purpose |
|---------|---------|
| `org.dcm4che3.image` | Image processing |
| `org.dcm4che3.imageio` | ImageIO integration |
| `org.dcm4che3.imageio.codec` | Compression codecs |

### Tools
| Package | Purpose |
|---------|---------|
| `org.dcm4che3.tool.findscu` | C-FIND SCU client |
| `org.dcm4che3.tool.storescu` | C-STORE SCU client |
| `org.dcm4che3.tool.jpg2dcm` | JPEG to DICOM converter |

### Services
| Package | Purpose |
|---------|---------|
| `org.dcm4che3.net.service` | SCP service implementations |
| `org.dcm4che3.net.pdu` | Protocol Data Units |
| `org.dcm4che3.media` | DICOMDIR support |

## Security Considerations

| Issue | Risk | Recommendation |
|-------|------|----------------|
| No TLS | High | Implement TLS 1.2+ for associations |
| No Authentication | High | Add AE title validation |
| No Encryption | High | Encrypt pixel data at rest |
| Expired Certificate | Medium | Renew signing certificate |
| eval() in JS Bridge | Medium | Use safe message parsing |
