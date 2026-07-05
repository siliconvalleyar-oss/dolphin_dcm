# DEX Classes Analysis - DolphinUSG

## Overview

Analysis of the `classes.dex` file from the DolphinUSG APK. The DEX contains 19,829 string entries, 2,646 types, 4,335 protos, and 13,875 methods.

## Package Distribution

### Application Code

| Package | Classes | Description |
|---------|---------|-------------|
| `com/sonostar/wirelessusg` | ~150 | Main app code |
| `com/sonoptek/measurekit` | ~30 | Measurement tools |
| `com/sonoptek/jni` | 1 | JNI bridge |

### Third-Party Libraries

| Package | Classes | Description |
|---------|---------|-------------|
| `org/dcm4che3/*` | ~100 | DICOM toolkit |
| `androidx/*` | ~500 | AndroidX libraries |
| `android/*` | ~300 | Android framework |
| `java/*` | ~200 | Java standard library |
| `org/json` | 5 | JSON parsing |
| `org/slf4j/*` | 10 | Logging |
| `pub/devrel/easypermissions` | 5 | Permissions helper |

### Obfuscated Code

| Package | Classes | Likely Purpose |
|---------|---------|---------------|
| `a/a/*` | ~100 | Utility classes |
| `a/b/*` | ~50 | Protocol handling |
| `a/c/*` | ~20 | Connection management |
| `a/d/*` | ~30 | DICOM implementation |
| `a/e/*` | ~10 | Error handling |
| `a/f/*` | ~80 | Network layer |
| `a/g/*` | ~15 | Image processing |
| `a/h/*` | ~10 | Configuration |
| `a/i/*` | ~20 | Data models |
| `a/j/*` | ~25 | Service layer |
| `a/k/*` | ~5 | Utilities |
| `a/l/*` | ~20 | Serialization |
| `b/a/a` | ~200 | DICOM library |
| `c/a/a/a` | ~17 | Secondary system |

## Class Analysis

### MainActivity (Core Scanning)

**Inner Classes**: 30+ anonymous/inner classes ($a through $z0)

| Inner Class | Likely Purpose |
|-------------|---------------|
| `MainActivity$a` | Connection handler |
| `MainActivity$b` | Stream processor |
| `MainActivity$c` | UI controller |
| `MainActivity$d` | Image buffer |
| `MainActivity$e` | Mode manager |
| `MainActivity$f` | Measurement handler |
| `MainActivity$g` | DICOM sender |
| `MainActivity$h` | Settings handler |
| `MainActivity$i` | Patient data |
| `MainActivity$j` | Report generator |
| `MainActivity$k` | File manager |
| `MainActivity$l` | Error handler |
| `MainActivity$m` | Thread manager |
| `MainActivity$n` | Timer handler |
| `MainActivity$o` | Color processor |
| `MainActivity$p` | PW processor |
| `MainActivity$q` | M-mode handler |
| `MainActivity$r` | Freeze handler |
| `MainActivity$s` | Capture handler |
| `MainActivity$t` | Export handler |
| `MainActivity$u` | Share handler |
| `MainActivity$v` | Help handler |
| `MainActivity$w` | Update handler |
| `MainActivity$x` | License handler |
| `MainActivity$y` | Analytics handler |
| `MainActivity$z` | Crash handler |
| `MainActivity$a0` through `$z0` | Additional handlers |

### Custom View Classes

| Class | Superclass | Purpose |
|-------|-----------|---------|
| `USBMImageView` | ImageView | B-Mode rendering |
| `USColorView` | View | Color Doppler |
| `USMotionView` | View | M-Mode |
| `USPWLineView` | View | PW line |
| `USPWExLineView` | View | Extended PW line |
| `USPWMotionView` | View | PW M-mode |
| `USPWExMotionView` | View | Extended PW M-mode |
| `USBiopsyView` | View | Biopsy guide |
| `USScaleView` | View | Scale overlay |
| `USMarkView` | View | Measurement marks |
| `LoopView` | View | Scroll selector |
| `USButton` | Button | Custom button |

### Measurement Kit Classes

| Class Range | Tools |
|-------------|-------|
| `a` - `e` | Distance measurement |
| `f` - `j` | Area measurement |
| `k` - `m` | Volume calculation |
| `n` - `p` | Angle measurement |
| `q` - `t` | Trace/perimeter |
| `u` - `z` | Doppler velocity |
| `a0` - `d0` | Heart rate/flow |

### DICOM Service Classes

| Class | Service |
|-------|---------|
| `BasicCEchoSCP` | C-ECHO |
| `BasicCFindSCP` | C-FIND |
| `BasicCStoreSCP` | C-STORE |
| `BasicCMoveSCP` | C-MOVE |
| `BasicCGetSCP` | C-GET |
| `BasicMPPSSCP` | MPPS |

## Method Analysis

### Key Method Signatures

```java
// Connection lifecycle
void onConnectionAccepted()
void onConnectionEstablished()
void onConnectionFailed()
void onConnectionRejected()
void onConnectionRejectedBlacklisted()

// UI operations
void freeze()
void loadImage()
void saveReport()
void measure()
void measure_volume()

// DICOM operations
void openFile(String path)
FileDescriptor openFileDescriptor(String path)
FileOutputStream openFileOutput(String path, int mode)
```

### String Statistics

| Category | Count | Examples |
|----------|-------|---------|
| DICOM UIDs | 500+ | Transfer syntaxes, SOP classes |
| Package names | 1000+ | Class references |
| Method names | 5000+ | Java methods |
| UI strings | 2000+ | Labels, messages |
| Config keys | 100+ | Settings keys |
| Error messages | 200+ | Exception text |
| Network strings | 50+ | URLs, ports |
| Chinese text | 10+ | UI localization |

## Obfuscation Mapping

### Likely Original Package Names

| Obfuscated | Likely Original |
|-----------|-----------------|
| `a/a` | `com.sonostar.utils` |
| `a/b` | `com.sonostar.protocol` |
| `a/c` | `com.sonostar.connection` |
| `a/d` | `com.sonostar.dicom` |
| `a/e` | `com.sonostar.error` |
| `a/f` | `com.sonostar.network` |
| `a/g` | `com.sonostar.image` |
| `a/h` | `com.sonostar.config` |
| `a/i` | `com.sonostar.model` |
| `a/j` | `com.sonostar.service` |
| `a/k` | `com.sonostar.util` |
| `a/l` | `com.sonostar.serialize` |
| `b/a/a` | `org.dcm4che3.wrapper` |
| `c/a/a/a` | `com.sonoptek.support` |

## Dependencies

### Compile Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| AndroidX AppCompat | 1.0+ | UI compatibility |
| AndroidX Core | 1.0+ | Core utilities |
| AndroidX Fragment | 1.0+ | Fragment support |
| AndroidX ConstraintLayout | 1.1+ | Layout |
| dcm4che3 | 3.x | DICOM |
| SLF4J | 1.7+ | Logging |
| EasyPermissions | 1.0+ | Runtime permissions |

### Native Dependencies

| Library | Architecture | Size |
|---------|-------------|------|
| libvessel_flow.so | arm64-v8a | 494 KB |
| libRSSupport.so | arm64-v8a | 1.2 MB |
| librs.*.so | arm64-v8a | ~600 KB |

## Build Configuration

### ProGuard Rules (Inferred)

```proguard
# Keep application classes
-keep class com.sonostar.wirelessusg.** { *; }
-keep class com.sonoptek.** { *; }

# Keep DICOM classes
-keep class org.dcm4che3.** { *; }

# Obfuscate everything else
-repackageclasses ''
-allowaccessmodification
```

### minSdkVersion

```gradle
minSdkVersion 23  // Android 6.0 (Marshmallow)
```

### targetSdkVersion

```gradle
targetSdkVersion 30  // Android 11
```
