# biometric-oemware-guide

**"Which firmware is inside my biometric terminal — and how do I integrate it?"**

A field guide to the OEM firmware families behind Indian-market (and Asian-market) biometric attendance terminals. Most of these devices are rebrands of a small number of reference designs. Once you know the firmware family, integration is straightforward.

---

## The problem

You bought a biometric terminal branded "Secureye", "eSSL", "Time India", "BioMax", or similar. The vendor software only runs on Windows. You want:
- Attendance data on your Linux server
- Door unlock without a Windows DLL
- ERPNext / Zoho / custom integration

The manufacturer's name on the box tells you almost nothing — what matters is the **firmware family** inside.

---

## Firmware families — quick identification

### Family 1: ZKTeco ZEM series (genuine ZKTeco)

**Identify by**: ZKTeco logo, firmware string like `Ver 6.70 Build 6.6.x`, platform `ZLM30_TFT`, device port 4370 returns real data with pyzk.

**Common devices**: ZKTeco X2008, F22, UA300, K40, G4, iClock series

**Integration path**: Port 4370 Pull protocol via [pyzk](https://github.com/fananimi/pyzk) or [python-zklib](https://github.com/dnaextrim/python_zklib). Door unlock via `zk.unlock()`.

**Toolkit**: → [zkteco-linux-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-linux-toolkit)

**Gotcha**: Communication Server must be enabled on the device first (`Menu → Comm → PC Connection`). Without it, pyzk returns "Unauthenticated".

---

### Family 2: ZKTeco ZAM series (newer ZKTeco OEMs — ADMS push only)

**Identify by**: Firmware platform `ZAM180_TFT` or `ZAM200_TFT`, face recognition (SpeedFace line), SSH on port 3718 (Dropbear), port 4370 returns `0x07f0` for all commands.

**Common devices / brands**: eSSL X2008, ZKTeco SpeedFace-V5L, SpeedFace-V4L, K40 Pro, many Indian OEMs (ESSL, BioMax, ESSLSECURITY branded face+finger devices).

**Integration path**: Pull protocol disabled. Use **ADMS HTTP push** — device sends attendance to your server. `Menu → COMM → Cloud Server Setting → ADMS`.

**Server**: → [adms-push-server](https://github.com/Vibhav-Aggarwal/adms-push-server)

**Gotcha**: Some firmware sends TLS even when HTTPS is set to Off. Physical menu + reboot required. See [troubleshooting](https://github.com/Vibhav-Aggarwal/adms-push-server/blob/main/docs/troubleshooting.md).

---

### Family 3: Realand / EBKN BioFace M61 firmware

**Identify by**: Firmware string `M61BH v3.x` in device menu → System Info, or `TerminalType=F500` in WebSocket login frame, product names like "S-FB3K", "BIOFACE M61BH", "EbknFace V3.0" / "EbknFinger V3.0" algorithm names.

**Common devices / brands**: Secureye S-FB3K, Realand BIOFACE M61BH, Realand A-C121, possibly other Realand/EBKN branded face terminals sold in India.

**Integration path**: Three modes in `Menu → Comm → Cloud Server`:
- LogClient (BioFace M61 XML/TCP, port 5005) — attendance only
- FkWeb (HTTP JSON POST to `/ebkn`) — attendance + command injection
- WebSocket (RFC6455 + XML, F500 protocol) — attendance + **door unlock** ✅

**Listener**: → [realand-bioface-m61-protocol](https://github.com/Vibhav-Aggarwal/realand-bioface-m61-protocol)

**Gotcha**: WebSocket mode requires full URL with scheme: `ws://192.168.1.1:8089`. Just `192.168.1.1` silently falls back to a proprietary vendor protocol (SBPCCOMM) that requires a Windows DLL.

---

### Family 4: Hikvision / Dahua (Video-security OEMs)

**Identify by**: ISAPI HTTP API on port 80, ONVIF, face recognition + access control in same unit.

**Integration**: ISAPI REST API (well-documented). Not covered here — see [Hikvision ISAPI](https://www.hikvision.com/en/support/download/sdk/).

---

### Family 5: Suprema (Korean, enterprise)

**Identify by**: BioStar 2 software, devices named BioEntry, BioLite, FaceStation.

**Integration**: BioStar 2 SDK / REST API. Well-documented by vendor.

---

## Decision tree

```
Your device has attendance + biometric terminal
│
├── ZKTeco logo OR ZKFinger algorithm?
│   ├── pyzk returns real data from port 4370?
│   │   └── YES → Family 1: zkteco-linux-toolkit
│   └── port 4370 returns 0x07f0 or empty?
│       └── Family 2: adms-push-server
│
├── Firmware string "M61BH" or "EbknFace/EbknFinger" in System Info?
│   └── Family 3: realand-bioface-m61-protocol
│
├── ISAPI / ONVIF on port 80?
│   └── Family 4: Hikvision — use ISAPI
│
└── Unknown — try each in order:
    1. nmap -p 4370,3718,80 <device-ip>
    2. Try pyzk with password=0 and password=123456
    3. Try GET /iclock/cdata.aspx (ADMS probe)
    4. Set device to "WebSocket" mode, check firmware version
```

---

## Indian market rebrand map

| Brand | Model | Firmware family | Notes |
|-------|-------|-----------------|-------|
| eSSL / ESSLSECURITY | X2008 | ZAM180 (Family 2) | OEM of ZKTeco SpeedFace-V5L |
| eSSL / ESSLSECURITY | X990, MB160 | ZEM (Family 1) | Older genuine ZKTeco OEM |
| Secureye | S-FB3K | Realand M61BH (Family 3) | EBKN firmware, 3 protocols |
| Secureye | S-FB4K | Realand M61BH (Family 3) | Sister model, same firmware |
| Time India | TA100 | ZEM (Family 1) | ZKTeco OEM |
| BioMax | BM-FA600 | ZAM (Family 2) likely | Verify by checking port 4370 |
| Realand | A-C series | Realand M61BH (Family 3) | Direct Realand brand |
| ZKTeco | X2008 | ZEM510 (Family 1) | Genuine ZKTeco |
| ZKTeco | SpeedFace-V5L | ZAM180 (Family 2) | Newer, Pull disabled |

> This table is empirical — if you find a different family for any model, open an issue.

---

## Protocol port reference

| Port | Protocol | Family |
|------|----------|--------|
| 4370 TCP/UDP | ZKTeco Pull SDK | 1, 2 (disabled on 2) |
| 23 TCP | Telnet shell | 1 (ZEM510) |
| 3718 TCP | Dropbear SSH | 2 (ZAM180, credentials unknown) |
| 80 TCP | ADMS `/iclock/*` or FkWeb `/ebkn` | 2, 3 |
| 5005 TCP | BioFace M61 LogClient (XML/TCP) | 3 |
| any TCP | WebSocket F500 (RFC6455 + XML) | 3 |
| 80, 443 | ISAPI REST | 4 |

---

## Resources

| Repo | Covers |
|------|--------|
| [adms-push-server](https://github.com/Vibhav-Aggarwal/adms-push-server) | ADMS HTTP push receiver (Family 2 + any ADMS device) |
| [realand-bioface-m61-protocol](https://github.com/Vibhav-Aggarwal/realand-bioface-m61-protocol) | Realand/EBKN BioFace M61 — all 3 protocols, door unlock |
| [zkteco-linux-toolkit](https://github.com/Vibhav-Aggarwal/zkteco-linux-toolkit) | ZKTeco port 4370 scripts, Zoho People sync |
| [secureye-s-fb3k-research](https://github.com/Vibhav-Aggarwal/secureye-s-fb3k-research) | Full reverse-engineering notes for Secureye S-FB3K |
| [pyzk](https://github.com/fananimi/pyzk) | Python ZKTeco SDK (Pull protocol) |
| [biometric_integration](https://github.com/KhaledBinAmir/biometric_integration) | Frappe/ERPNext app for ADMS devices |
| [ZKTeco PUSH Protocol v2.0](https://www.scribd.com/document/695654988/PUSH-SDK-Communication-Protocol-V2-0-1) | Official ADMS wire format spec |

---

## Contributing

If you've integrated a device not listed here, open a PR adding it to the rebrand map with:
- Brand + model
- Firmware string (from System Info menu)
- Which family it belongs to
- Any gotchas
