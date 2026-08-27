# Terminal Firmware Atlas

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Schema.org](https://img.shields.io/badge/Schema.org-CodeMeta_JSON--LD-orange.svg)](codemeta.json)
[![Citation](https://img.shields.io/badge/Citation-CITATION.cff-green.svg)](CITATION.cff)
[![Author](https://img.shields.io/badge/ORCID-0009--0000--7686--7119-green.svg)](https://orcid.org/0009-0000-7686-7119)

> **"Which firmware is inside my biometric terminal — and how do I integrate it on Linux without vendor lock-in?"**

The **Terminal Firmware Atlas** is an open hardware identification matrix, OEM taxonomy, and integration decision guide for biometric attendance and access control terminals (Secureye, eSSL, Realand, BioMax, ZKTeco, Time India).

Most biometric devices sold in Asian and Indian markets are rebrands of a small number of reference OEM firmware designs. Once you identify the underlying firmware family, Linux server integration and door access automation are straightforward.

---

## 🏛️ Ecosystem Suite Map

```
                               ┌─────────────────────────────────────────┐
                               │        terminal-firmware-atlas          │
                               │  (Hardware Taxonomy & Decision Engine)  │
                               └────────────────────┬────────────────────┘
                                                    │
             ┌──────────────────────────────────────┼──────────────────────────────────────┐
             │                                      │                                      │
             ▼                                      ▼                                      ▼
┌─────────────────────────┐            ┌─────────────────────────┐            ┌─────────────────────────┐
│    ebkn-m61-protocol    │            │    zkteco-adms-server   │            │   zkteco-pull-toolkit   │
│  (EBKN/Realand M61BH)   │            │   (ZKTeco Push Mode)    │            │   (ZKTeco Pull Mode)    │
│  Specs + Listener + Door│            │  FastAPI Push Receiver  │            │ Direct UDP/Port 4370 CLI│
└─────────────────────────┘            └─────────────────────────┘            └─────────────────────────┘
```

---

## 🔍 Firmware Families & Quick Identification

### Family 1: ZKTeco ZEM Series (Genuine ZKTeco / Older OEMs)
* **Identify by:** ZKTeco logo, firmware string like `Ver 6.70 Build 6.6.x`, platform `ZLM30_TFT`, port 4370 returns real data.
* **Common Devices:** ZKTeco X2008, F22, UA300, K40, G4, iClock series, older eSSL (X990, MB160).
* **Integration Path:** Port 4370 UDP/TCP Pull protocol.
* **Toolkit:** 👉 [zkteco-pull-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-pull-toolkit)

---

### Family 2: ZKTeco ZAM Series (Newer Face/SpeedFace OEMs — Push Only)
* **Identify by:** Platform `ZAM180_TFT` or `ZAM200_TFT`, face recognition, Dropbear SSH on port 3718, port 4370 returns `0x07f0` (Pull disabled).
* **Common Devices:** eSSL X2008, ZKTeco SpeedFace-V5L, SpeedFace-V4L, K40 Pro, BioMax BM-FA600.
* **Integration Path:** Inbound **ADMS HTTP Push** (`Menu → COMM → Cloud Server Setting → ADMS`).
* **Server:** 👉 [zkteco-adms-server](https://github.com/Vibhav-Aggarwal/zkteco-adms-server)

---

### Family 3: EBKN / Realand BioFace M61 Series
* **Identify by:** Firmware string `M61BH v3.x` in System Info, `TerminalType=F500` in WebSocket login frame, algorithm `EbknFace V3.0`.
* **Common Devices:** Secureye S-FB3K, Secureye S-FB4K, Realand BIOFACE M61BH, Realand A-C series.
* **Integration Path:** 3 modes selectable in `Menu → Comm → Cloud Server`:
  * **LogClient** (XML/TCP on port 5005) — attendance logs
  * **FkWeb** (HTTP JSON POST to `/ebkn`) — attendance + command injection
  * **WebSocket F500** (RFC6455 + XML) — attendance + **door unlock**
* **Reference Listener & Specs:** 👉 [ebkn-m61-protocol](https://github.com/Vibhav-Aggarwal/ebkn-m61-protocol)

---

### Family 4: Hikvision / Dahua (Video Access Control)
* **Identify by:** ISAPI HTTP API on port 80/443, ONVIF.
* **Integration:** ISAPI REST API (standard vendor documentation).

---

### Family 5: Suprema (Enterprise Access)
* **Identify by:** BioStar 2 software, devices named BioEntry, BioLite, FaceStation.
* **Integration:** BioStar 2 REST API & C SDK.

---

## 🧭 Identification Decision Tree

```text
Your Biometric Attendance Terminal
│
├── ZKTeco logo OR ZKFinger / ZKFace algorithm?
│   ├── pyzk returns real data from port 4370?
│   │   └── YES ──> Family 1: zkteco-pull-toolkit (Port 4370 Pull)
│   └── port 4370 returns 0x07f0 or empty?
│       └── YES ──> Family 2: zkteco-adms-server (ADMS HTTP Push)
│
├── Firmware string "M61BH" or "EbknFace/EbknFinger" in System Info?
│   └── YES ──> Family 3: ebkn-m61-protocol (LogClient / FkWeb / WebSocket F500)
│
├── ISAPI / ONVIF on port 80/443?
│   └── YES ──> Family 4: Hikvision ISAPI
│
└── Unknown Device? 
    1. Scan ports: nmap -p 23,80,3718,4370,5005 <device-ip>
    2. Probe ADMS: curl "http://<device-ip>/iclock/cdata.aspx?SN=test"
    3. Test WebSocket mode with a test listener
    4. Open an issue with your findings!
```

---

## 🗺️ Indian Market Rebrand Mapping

| Brand Label | Model Name | Real OEM / Firmware Family | Primary Protocol | Verified Suite Repo |
| :--- | :--- | :--- | :--- | :--- |
| **Secureye** | S-FB3K | EBKN / Realand M61BH (Family 3) | WebSocket / FkWeb | [ebkn-m61-protocol](https://github.com/Vibhav-Aggarwal/ebkn-m61-protocol) |
| **Secureye** | S-FB4K | EBKN / Realand M61BH (Family 3) | WebSocket / FkWeb | [ebkn-m61-protocol](https://github.com/Vibhav-Aggarwal/ebkn-m61-protocol) |
| **Realand** | BIOFACE M61BH | EBKN / Realand M61BH (Family 3) | LogClient / WS | [ebkn-m61-protocol](https://github.com/Vibhav-Aggarwal/ebkn-m61-protocol) |
| **eSSL** | X2008 (Face) | ZKTeco ZAM180 (Family 2) | ADMS Push | [zkteco-adms-server](https://github.com/Vibhav-Aggarwal/zkteco-adms-server) |
| **ZKTeco** | SpeedFace-V5L | ZKTeco ZAM180 (Family 2) | ADMS Push | [zkteco-adms-server](https://github.com/Vibhav-Aggarwal/zkteco-adms-server) |
| **ZKTeco** | X2008 (Finger) | ZKTeco ZEM510 (Family 1) | Port 4370 Pull | [zkteco-pull-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-pull-toolkit) |
| **eSSL** | X990, MB160 | ZKTeco ZEM (Family 1) | Port 4370 Pull | [zkteco-pull-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-pull-toolkit) |
| **Time India** | TA100 | ZKTeco ZEM (Family 1) | Port 4370 Pull | [zkteco-pull-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-pull-toolkit) |
| **BioMax** | BM-FA600 | ZKTeco ZAM (Family 2 likely) | ADMS Push | [zkteco-adms-server](https://github.com/Vibhav-Aggarwal/zkteco-adms-server) |

---

## 📦 Suite Repositories & Resources

| Repository | Role & Coverage |
| :--- | :--- |
| **[ebkn-m61-protocol](https://github.com/Vibhav-Aggarwal/ebkn-m61-protocol)** | EBKN / Realand BioFace M61 — all 3 protocol wire specifications, Python reference listener, and door unlock. |
| **[zkteco-adms-server](https://github.com/Vibhav-Aggarwal/zkteco-adms-server)** | Standalone Python ADMS push receiver (ZKTeco ZAM series, SpeedFace, eSSL). |
| **[zkteco-pull-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-pull-toolkit)** | Python scripts for ZKTeco port 4370 direct UDP/TCP pull, CSV export, time sync, and Zoho People sync. |
| **[pyzk](https://github.com/fananimi/pyzk)** | Core Python library for ZKTeco port 4370 Pull protocol. |
| **[biometric_integration](https://github.com/KhaledBinAmir/biometric_integration)** | Frappe / ERPNext integration application for ADMS devices. |

---

## 🤝 Contributing

Have you reverse-engineered or integrated a device not listed here? Please submit a **[Device Report Issue](https://github.com/Vibhav-Aggarwal/terminal-firmware-atlas/issues/new?template=device_report.md)** or open a PR adding it to the rebrand matrix with:
1. Brand and physical model
2. Firmware string (from `Menu → System Info`)
3. Network scan & observed protocol behavior
4. Any quirks or gotchas

---

## 📄 Citation & License

- **License:** [Apache-2.0](LICENSE) (Code) / [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/) (Documentation)
- **Schema.org:** Machine-readable metadata in [`codemeta.json`](codemeta.json)
- **Citation:** See [`CITATION.cff`](CITATION.cff) or cite as:
  ```bibtex
  @software{Aggarwal_Terminal_Firmware_Atlas_2026,
    author = {Aggarwal, Vibhav},
    title = {{Terminal Firmware Atlas: Open Identification Matrix & Architecture Reference for Biometric Devices}},
    url = {https://github.com/Vibhav-Aggarwal/terminal-firmware-atlas},
    year = {2026}
  }
  ```
