# MotoLK Studio v1.0.0 📱

**The Ultimate Native Motorola MediaTek LK Bootloader Toolkit & Patcher**

---

## 🌟 Overview

**MotoLK Studio** is a professional-grade, high-performance C++ application with a modern Qt GUI designed for reverse engineers, firmware developers, and technicians working on Motorola MediaTek (MTK) bootloaders. Developed by **Extra-Team**, this tool enables precise, deterministic patching of LK boot images while maintaining cryptographic integrity and hardware compatibility.

**Current Version:** v1.0.0 | **Platform:** Windows 10/11 (64-bit) | **Architecture:** AArch64 & ARM32 Thumb2

---

## ✨ Core Features

### 🔓 **In-Place Security Gate Injection**
- **Compact, bit-identical runtime verification stubs** for both architectures:
  - **AArch64:** 584-byte injection
  - **ARM32 Thumb2:** 416-byte injection
- **Full Pointer Authentication (PA) preservation** — maintains PACIASP/PACIBSP instructions to prevent kernel panics and boot loops
- **Hardware-aware code generation** — emits native ARM/Thumb instructions with zero manual assembly errors

### 🛠️ **One-Click Patch Presets**
- **unlock-serial** 🔓 — Inject runtime serial validation gate accepting custom unlock tokens
- **erase-serial** 🗑️ — Allow erasing protected partitions (FRP, nvdata, etc.) via fastboot
- **factory-allow** 🏭 — Expose hidden diagnostic OEM commands for authorized servicing
- **full-allow** 🌟 — Combine all security overrides in a single application

### 🔑 **Deterministic Key Generation Engine**
- **100% parity with Python reference implementation** — identical mathematical RNG
- **Multiple derivation schemes:**
  - Serial Number–based keys
  - Device ID (e.g., nevada)–based keys
  - 15-digit IMEI–based keys
  - Protected Partition name–based keys
- **Instant clipboard export** — alphanumeric (alnum) or ASCII modes

### 🔏 **Automated CERT2 Resigning & ASN.1 DER Parsing**
- **Full LibLK image lifecycle:** unpack → modify → repack → sign
- **Automatic SHA256 integrity recalculation** for all boot image sections
- **Bit-identical MTK CERT2 cryptographic signing** — ensures signed binaries match reference implementations
- **ASN.1 DER parsing** for certificate inspection and validation

### 📱 **4-Stage Smart Fastboot Manager**
- **Automatic device detection** — identifies connected devices, active slot (A/B), serial number, and security status
- **Guarded flashing safety:**
  - Flashing disabled until patched image passes cryptographic validation
  - Safety warnings encourage Slot B testing first
  - Real-time validation feedback
- **Batch operations** — flash multiple partitions in sequence with rollback protection

### 🔍 **Interactive Hex Diff Viewer**
- **Real-time side-by-side binary comparison** — original vs. patched bytes with color highlighting
- **Offset mapping and jump navigation** — quickly locate changes in large binaries
- **Report export** — generate detailed patch summaries for documentation

---

## 🏗️ Project Architecture

### Directory Structure

```
ValProtocol_MotoLKStudio/
├── ValProtocolMotoLKStudio.pro          ← Main qmake project file
├── src/                                  ← Source code root
│   ├── main.cpp                          ← GUI app entry point
│   ├── cli_main.cpp                      ← CLI entry point (headless mode)
│   │
│   ├── core/                             ← Core utilities (no external deps)
│   │   ├── types.h                       ← Type definitions, enums
│   │   ├── utils.h / utils.cpp           ← Helper functions, binary ops
│   │   └── logger.h / logger.cpp         ← Structured logging system
│   │
│   ├── liblk/                            ← LK boot image handling
│   │   ├── lk_structures.h               ← MTK LK format structures
│   │   ├── lk_image.h / lk_image.cpp     ← Parse & validate LK images
│   │   └── lk_repacker.h / lk_repacker.cpp ← Repack with correct headers
│   │
│   ├── crypto/                           ← Cryptography operations
│   │   ├── sha_helper.h / sha_helper.cpp ← SHA1/SHA256 hashing
│   │   ├── asn1_der.h / asn1_der.cpp     ← ASN.1 DER codec
│   │   ├── mtk_cert.h / mtk_cert.cpp     ← MTK certificate handling
│   │   └── keygen.h / keygen.cpp         ← Key derivation engine
│   │
│   ├── device/                           ← Device communication
│   │   └── fastboot_manager.h / fastboot_manager.cpp ← Fastboot protocol
│   │
│   ├── analyzer/                         ← Code analysis (requires Capstone)
│   │   ├── capstone_engine.h / capstone_engine.cpp    ← Disassembler wrapper
│   │   ├── gate_discovery.h / gate_discovery.cpp      ← Find security gates
│   │   └── function_prologue.h / function_prologue.cpp ← Detect entry points
│   │
│   ├── patcher/                          ← Code patching (requires Capstone)
│   │   ├── arm64_emitter.h / arm64_emitter.cpp        ← ARM64 codegen
│   │   ├── thumb_emitter.h / thumb_emitter.cpp        ← Thumb-2 codegen
│   │   ├── known_builds.h / known_builds.cpp          ← Firmware signatures
│   │   └── patch_engine.h / patch_engine.cpp          ← Patch application
│   │
│   ├── pipeline/                         ← Processing pipeline
│   │   └── lk_pipeline.h / lk_pipeline.cpp ← Orchestrate load→analyze→patch→verify
│   │
│   └── ui/                               ← Qt GUI (requires Qt Widgets)
│       ├── main_window.h / main_window.cpp      ← Main window, menu, tabs
│       ├── patch_worker.h / patch_worker.cpp    ← Worker thread for long ops
│       └── components/                          ← Reusable widgets
│           ├── log_console.h / log_console.cpp  ← Log display
│           ├── keygen_widget.h / keygen_widget.cpp ← Key generation UI
│           └── hex_diff_dialog.h / hex_diff_dialog.cpp ← Binary diff viewer
│
├── resources/                            ← Application resources
│   ├── resources.qrc                     ← Qt resource manifest
│   └── app.rc                            ← Windows icon & version info
│
├── third_party/                          ← External dependencies
│   └── capstone/                         ← Capstone disassembler
│       ├── include/capstone.h            ← C API headers
│       └── lib/capstone.dll              ← Windows binary
│
└── build/                                ← Build output (generated)
    ├── debug/                            ← Debug build
    └── release/                          ← Release build
```

### Module Dependencies

```
Core (types, utils, logger) — NO external dependencies
    ↓
Crypto (SHA, ASN.1, certs, keygen) — depends on Core
    ↓
LibLK (image parser, repacker) — depends on Core + Crypto
    ↓
├─→ Device (Fastboot) — depends on Core
│
├─→ [CONDITIONAL: build_engine]
│   ├─→ Analyzer (Capstone, gate discovery, prologue detection)
│   ├─→ Patcher (ARM64/Thumb emitters, known builds, patch engine)
│   └─→ Pipeline (orchestrates all above)
│
└─→ [CONDITIONAL: build_gui]
    └─→ UI (main window, keygen, hex diff, log console) — depends on all above
```

---

## 📋 Supported Devices & Chipsets

### ✅ Motorola MediaTek Smartphones

**Moto G Series:**
- Moto G Power 5G (2023, 2024)
- Moto G 5G (2023, 2024)
- Moto G Play (2023, 2024)
- Moto G73 5G, G54 5G, G53, G13, G23
- Moto G Pure, Moto G Stylus (MTK editions)

**Moto E Series:**
- Moto E13, E22, E22i, E20, E7, E7 Power, E6

**Moto Edge Series (MTK variants):**
- Moto Edge 40 Neo
- Moto Edge 50 Fusion (MTK)
- Moto Edge Lite

### ✅ Supported MediaTek SoCs

**Dimensity:** 7020, 7025, 930, 8020, 1050, 700, 810 (MT6855, MT6877, MT6833, MT6893...)

**Helio:** G99, G88, G85, G37, G35, G25, P35, P22 (MT6769, MT6768, MT6765, MT6762...)

---

## ⚙️ Build Requirements & Configuration

### Minimum Requirements
- **OS:** Windows 10 / 11 (64-bit)
- **Compiler:** MSVC 2019+ (C++17 support required)
- **Build System:** Qt Creator with qmake
- **Qt Version:** Qt 5.15 or later (Qt Widgets module)

### Configurable Build Flags

```bash
# Enable/disable major components via CONFIG in .pro file or command line:
qmake CONFIG+=build_engine CONFIG+=build_gui CONFIG+=build_tests

# Available flags:
# - build_engine    Build analyzer/patcher engine (requires Capstone)
# - build_gui       Build Qt GUI application
# - build_tests     Build unit test suite
# - c++17           Enable C++17 standard
```

### Dependencies

| Component | Status | Notes |
|-----------|--------|-------|
| **Capstone** | Optional | Required for `build_engine`; binary located at `third_party/capstone/lib/capstone.dll` |
| **Qt Widgets** | Optional | Required for `build_gui`; install via Qt installer |
| **MSVC Runtime** | Required | Included with Visual Studio or redistributable |

---

## 🚀 Getting Started

### 1. Clone / Extract Project
```bash
cd ValProtocol_MotoLKStudio
```

### 2. Open in Qt Creator
```bash
# Via command line:
qtcreator ValProtocol_MotoLKStudio.pro

# Or open Qt Creator, then File → Open Project → select .pro file
```

### 3. Configure Build Variant
In **Projects** panel → **Build Settings** → **qmake** section, set:
```
CONFIG += build_gui build_engine build_tests c++17
```

### 4. Build
```bash
# In Qt Creator: Ctrl+B (or Build → Build Project)
# Or via command line:
qmake
nmake  # or mingw32-make if using MinGW
```

### 5. Run
```bash
# Debug: ./build/debug/MotoLKStudio.exe
# Release: ./build/release/MotoLKStudio.exe
```

---

## 🔧 Usage Examples

### GUI Mode
1. **Load LK Image** — File → Open, select a Motorola MTK bootloader (usually `lk.bin` or `logo.bin`)
2. **Select Patch Preset** — Choose from unlock-serial, erase-serial, factory-allow, or full-allow
3. **Review Changes** — Hex Diff Viewer shows before/after binary comparison
4. **Generate Unlock Tokens** (if needed) — Use Keygen Widget with Serial Number or IMEI
5. **Validate Patch** — Cryptographic signature check performed automatically
6. **Flash to Device** — Connect device via USB, enable Fastboot mode, click Flash

### CLI Mode (Headless)
```bash
MotoLKStudio.exe --cli --input lk.bin --preset unlock-serial --output lk-patched.bin
MotoLKStudio.exe --cli --input lk.bin --keygen --serial 12345678 --output key.txt
```

---

## 🧪 Testing

### Run Unit Tests
```bash
qmake CONFIG+=build_tests
nmake
./build/debug/tests.exe
```

**Test Suites Included:**
- **Golden Parity Tests** — Verify patching produces bit-identical results
- **Crypto Tests** — Validate SHA256, ASN.1 DER, certificate signing
- **LK Image Tests** — Parse/repack integrity checks

---

## 📊 Technical Highlights

### Performance
- **Sub-second patch generation** for typical 512 KB – 2 MB boot images
- **Parallel Capstone disassembly** (if multi-threaded analyzer enabled)
- **Real-time hex diff** with scroll-linked comparison views

### Security
- **No external network calls** — fully offline operation
- **Cryptographic validation** before flashing prevents bricked devices
- **Immutable patch presets** — no arbitrary code injection possible
- **Audit logs** — all operations logged with timestamps

### Compatibility
- **Bit-identical output** — ensures patched images are reproducible
- **Pointer Authentication preservation** — hardware security features maintained
- **Multi-slot support** — automatic A/B partition detection and management

---

## 🔗 Links & Contact

**Project Repository:**  
[GitHub / GitLab - Extra-Team](https://github.com/extra-team)

**Community & Updates:**  
📢 [Telegram: Extra Developers](https://t.me/ExtraDevlopers/65)

**For Full Source Code & Professional Inquiries:**  
💬 [Telegram: Mohammed Silan](https://t.me/Mohammedsilan)

---

## 📝 License

**MotoLK Studio v1.0.0** — Developed by **Extra-Team**  
Distributed for educational, research, and authorized professional use only.

---

## ⚠️ Disclaimer

This tool is designed for **authorized firmware research and legitimate device servicing only**. Users are responsible for:
- Obtaining proper authorization from device owners
- Complying with local laws and regulations
- Understanding the risks of bootloader modification
- Creating backups before flashing

**Extra-Team** assumes no liability for data loss, device damage, or legal consequences arising from misuse.

---

**Last Updated:** September 2026  
**Maintained by:** Extra-Team  
**Status:** Production Ready ✅
