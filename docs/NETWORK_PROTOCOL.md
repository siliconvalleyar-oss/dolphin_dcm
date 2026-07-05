# Network Protocol - DolphinUSG

## Overview

DolphinUSG uses multiple network protocols for communication with the wireless ultrasound probe, DICOM PACS servers, and Modality Worklist servers.

## Connection Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         INTERNET                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
            ┌───────────────┐   ┌───────────────┐
            │  PACS Server  │   │  MWL Server   │
            │  (DICOM)      │   │  (DICOM)      │
            └───────┬───────┘   └───────┬───────┘
                    │                   │
                    │  DICOM Protocol   │
                    │  (TCP/IP)         │
                    │                   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   DolphinUSG App  │
                    │   (Android)       │
                    └─────────┬─────────┘
                              │
                              │  WiFi/USB
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
            ┌───────────────┐   ┌───────────────┐
            │  US Probe     │   │  US Probe     │
            │  (WiFi)       │   │  (USB)        │
            └───────────────┘   └───────────────┘
```

## 1. Probe Connection

### WiFi Connection

| Parameter | Value |
|-----------|-------|
| Protocol | TCP/UDP |
| Default Gateway | 192.168.1.1 |
| Settings Intent | `android.settings.WIFI_SETTINGS` |
| Permission | `com.sonoptek.USB_PERMISSION` |

### Connection States

```
┌──────────────┐
│  Disconnected│ ←─── Initial state
└──────┬───────┘
       │ Connect
       ▼
┌──────────────┐
│  Connecting  │ ←─── WiFi association
└──────┬───────┘
       │
       ├──────────────────┐
       │                  │
       ▼ Success          ▼ Failure
┌──────────────┐   ┌──────────────┐
│  Connected   │   │  Failed      │
└──────┬───────┘   └──────────────┘
       │
       │ Stream Start
       ▼
┌──────────────┐
│  Streaming   │ ←─── Live imaging
└──────┬───────┘
       │
       │ Disconnect
       ▼
┌──────────────┐
│  Disconnected│
└──────────────┘
```

### Data Stream

```
Probe Frame → WiFi → Frame Buffer → Process → Display
    │
    └── Frame Structure:
        ├── Header (mode, timestamp, size)
        ├── B-Mode Data
        ├── Color Data (if Color mode)
        ├── PW Data (if PW mode)
        └── Metadata (gain, DR, etc.)
```

## 2. DICOM Protocol

### Connection Parameters

| Parameter | Configuration Key |
|-----------|-------------------|
| Local AE Title | `DCM1_THE_AE_TITLE` |
| Remote AE Title | `DCM2_THE_AE_TITLE` |
| Remote Port | `DCM2_PORT` |
| MWL AE Title | `MWL_THE_AE_TITLE` |
| MWL Port | `MWL_PORT` |

### TCP/IP Configuration

| Setting | Value |
|---------|-------|
| Protocol | TCP |
| Socket Timeout | Configurable |
| Buffer Size | Configurable |
| Keep Alive | Enabled |

### Association Flow

```
Client (App)                          Server (PACS)
     │                                      │
     │──── A-ASSOCIATE-RQ ────────────────►│
     │     (AE titles, contexts)           │
     │                                      │
     │◄─── A-ASSOCIATE-AC ────────────────│
     │     (Accepted contexts)             │
     │                                      │
     │──── DIMSE Command ─────────────────►│
     │     (C-STORE, C-FIND, etc.)         │
     │                                      │
     │◄─── DIMSE Response ────────────────│
     │     (Status, results)               │
     │                                      │
     │──── A-RELEASE-RQ ──────────────────►│
     │                                      │
     │◄─── A-RELEASE-RP ──────────────────│
     │                                      │
```

### Presentation Contexts

| Abstract Syntax | Transfer Syntax | Role |
|-----------------|-----------------|------|
| CT Image Storage | Explicit VR LE | SCU |
| US Image Storage | Explicit VR LE | SCU |
| MWL Information Model | Explicit VR LE | SCU |
| Verification | Implicit VR LE | SCP/SCU |

## 3. DICOM Services

### C-FIND (Query)

```
Request:
  PatientName: "DOE^JOHN"
  PatientID: ""
  StudyDate: "20240101-20241231"
  Modality: "US"

Response (Pending):
  PatientName: "DOE^JOHN"
  PatientID: "12345"
  StudyDate: "20240115"
  StudyDescription: "Abdomen US"

Response (Success):
  Status: 0x0000 (Success)
```

### C-STORE (Store)

```
Request:
  SOP Class: 1.2.840.10008.5.1.4.1.1.2
  SOP Instance: [generated UID]
  Pixel Data: [image data]

Response:
  Status: 0x0000 (Success)
  // or
  Status: 0xA700 (Out of resources)
```

### C-MOVE (Retrieve)

```
Request:
  SOP Class: 1.2.840.10008.5.1.4.1.2.1.2
  PatientName: "DOE^JOHN"
  StudyDate: "20240115"

Response (Pending):
  Dataset: [patient info]
  Status: 0xFF00 (Pending)

Response (Success):
  Status: 0x0000 (Success)
  // Sub-operations: 10/10 completed
```

## 4. UDP Protocol

### SYSLOG UDP

| Feature | Description |
|---------|-------------|
| Port | Configurable |
| Purpose | System logging |
| Format | Standard syslog |

### UDP Handling

```java
// UDP Reception
onReceive(UDPDatagram datagram) {
    // Process syslog message
    // Log to file
}

// Error Handling
Exception: "Cannot start UDP listener on {port}"
Exception: "Ignore UDP datagram from blacklisted {host}"
Exception: "Exception processing UDP received from {host}"
```

## 5. Connection Lifecycle

### Callbacks

| Callback | Trigger | Action |
|----------|---------|--------|
| `onConnectionAccepted` | Remote accepts association | Ready for commands |
| `onConnectionEstablished` | TCP connection established | Low-level connection ready |
| `onConnectionFailed` | Connection error | Retry or abort |
| `onConnectionRejected` | Association rejected | Check AE titles |
| `onConnectionRejectedBlacklisted` | Blacklisted peer | Log and reject |

### Error Handling

| Error | Code | Action |
|-------|------|--------|
| AE Title Not Recognized | 3/7 | Check configuration |
| Protocol Version Mismatch | - | Update software |
| Transfer Syntax Rejected | - | Negotiate alternative |
| Out of Resources | 0xA700 | Retry later |

## 6. Port Assignments

### DICOM Ports

| Service | Default Port | Description |
|---------|-------------|-------------|
| DICOM SCP | 11112 | Standard DICOM |
| DCM2 | Configurable | PACS server port |
| MWL | Configurable | Worklist server |

### Application Ports

| Service | Port | Description |
|---------|------|-------------|
| Probe Stream | Configurable | Ultrasound data |
| Control | Configurable | Probe commands |

## 7. Security Considerations

### Current State

| Issue | Risk | Status |
|-------|------|--------|
| No TLS | High | Vulnerable |
| No Authentication | High | Vulnerable |
| No Encryption | High | Vulnerable |
| Plain TCP | High | Vulnerable |

### Recommendations

| Enhancement | Priority | Implementation |
|-------------|----------|----------------|
| TLS 1.2+ | Critical | Add SSLContext |
| Certificate Pinning | High | Add TrustManager |
| Authentication | High | Add SCU credentials |
| Encryption at Rest | Medium | Add file encryption |

## 8. Network Monitoring

### Connection Metrics

| Metric | Description |
|--------|-------------|
| Latency | Round-trip time |
| Throughput | Data rate |
| Packet Loss | Lost frames |
| Reconnections | Connection stability |

### Logging

```
Start TCP Listener on {port}
Stop TCP Listener on {port}
Start UDP listener on {port}
Stop UDP listener on {port}
Received UDP datagram from {host}
```
