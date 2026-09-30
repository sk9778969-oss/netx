# NetX v1.0.3 — Release Guide

This document details the release packaging, deployment instructions, and clean-machine verification procedure for the standalone **NetX.exe** Windows application (formerly NetMonitor).

---

## 1. Release Overview

- **Application Name**: NetX (formerly NetMonitor)
- **Version**: 1.0.3
- **Target OS**: Windows 10 / Windows 11 (64-bit)
- **Distribution Format**: Standalone Single-File Executable (`NetX.exe`) & Release ZIP (`NetX-v1.0.3-Windows.zip`)
- **Generated Location**: `dist\NetX.exe` and `dist\NetX-v1.0.3-Windows.zip`
- **Output Size**: Approximately ~15.6 MB (contains bundled Python runtime, Scapy, Flask, Werkzeug, psutil, SQLite, HTML5 templates, and offline Chart.js bundle)

---

## 2. System Requirements & External Dependencies

### For the End User (Running NetX.exe):
1. **Operating System**: Windows 10 or Windows 11 (64-bit).
2. **Python**: **NOT REQUIRED**. Python runtime is embedded inside the executable.
3. **Npcap Packet Driver**:
   - **For LIVE Mode**: **REQUIRED**. Download and install from [https://npcap.com/#download](https://npcap.com/#download) with the option *"Install Npcap in WinPcap API-Compatible Mode"* checked.
   - **For DEMO Mode**: **NOT REQUIRED**. All 6 synthetic traffic scenarios run out-of-the-box without Npcap or administrator privileges.
   - **Why Npcap is Not Bundled**:
     - Npcap is a privileged Windows kernel-mode filter driver (`.sys`), not a user-space Python library that PyInstaller packages.
     - Npcap is licensed under the Insecure.Com LLC Free / OEM License, which prohibits redistribution or silent bundling with third-party software without an OEM redistribution agreement.
     - Silently downloading or installing privileged drivers violates Windows security policies and user trust.
4. **Permissions & Administrator Privileges**:
   - **Driver Installation**: Administrator privileges are required by Windows to install the Npcap kernel filter driver.
   - **Live Capture**: If Npcap was installed with the option *"Restrict Npcap driver's access to Administrators only"*, NetX must be launched using **Run as administrator** to capture physical frames.
   - **Demo Mode**: Requires standard user privileges only.

---

## 3. Release Packaging & Distribution Strategy

### Distribution ZIP Layout:
The release archive `dist\NetX-v1.0.3-Windows.zip` packages the standalone binary alongside essential user documentation:

```text
NetX-v1.0.3-Windows.zip
├── NetX.exe                    (Standalone portable application)
├── README.txt                  (Quick-start guide and system requirements)
└── Npcap_Installation.txt      (Step-by-step driver setup instructions)
```

While `NetX.exe` is completely portable and contains all application dependencies (Flask, Scapy, SQLite, templates, Chart.js), Npcap is an external kernel driver. To ensure users have a smooth out-of-the-box experience, NetX provides:

1. **In-App First-Run Setup Modal**: When launched without Npcap, the web dashboard opens immediately and presents an interactive setup modal with direct download link, step-by-step instructions, and an on-the-fly **Recheck Status** button.
2. **Instant Demo Exploration**: Users can evaluate all telemetry graphs, flow tables, and explainable anomaly detection immediately in Demo Mode without installing any drivers.

---

## 4. End-User Workflow & First-Run Setup Experience

When an end user downloads and launches `NetX.exe`:

```text
User launches NetX.exe
        ↓
NetX checks single-instance Named Mutex (Local\NetX_Application_Lock_2026)
        ↓
NetX performs startup diagnostics
        ↓
Initializes %LOCALAPPDATA%\NetX\{data, logs, reports}
(safely copies legacy %LOCALAPPDATA%\NetMonitor data if present)
        ↓
Discovers network adapters & checks Npcap live-capture backend
        ↓
Allocates local HTTP port (5000 or next free)
        ↓
Opens default browser (http://127.0.0.1:<port>)
        ↓
┌────────────────────────────────────────┴────────────────────────────────────────┐
│                                                                                 │
▼                                                                                 ▼
[Npcap Detected & Operational]                                    [Npcap Missing or Unusable]
        ↓                                                                 ↓
Dashboard READY for LIVE capture                          First-Run Setup Modal opens automatically
        ↓                                                                 ↓
Select adapter & click START LIVE                         Options presented to user:
                                                          1. Click "Install Npcap" (opens https://npcap.com/#download)
                                                          2. Install with WinPcap compatibility enabled
                                                          3. Click "Recheck Status" (dynamically activates LIVE mode)
                                                             — OR —
                                                          4. Click "Start Demo Mode" (explore immediately without driver)
```

---

## 5. Live Mode vs. Demo Mode Isolation

NetX enforces strict data boundary separation:

| Attribute | LIVE Mode | DEMO Mode |
| :--- | :--- | :--- |
| **Data Source** | Physical NIC (Wi-Fi/Ethernet) via Npcap | Mathematical traffic synthesis engine |
| **Npcap Required** | Yes | No |
| **Database** | `%LOCALAPPDATA%\NetX\data\netx.db` | `%LOCALAPPDATA%\NetX\data\netx_demo.db` |
| **Telemetry History** | Real network packets and active flows | Purely synthetic test vectors |
| **Cross-Contamination**| Zero (separate database and memory buffers) | Zero (cleared on demo start) |

---

## 6. Clean-Machine Verification Checklist

To verify that `NetX.exe` works on a completely clean Windows machine without Python or developer tools:

1. **Copy Binary**: Transfer only `dist\NetX.exe` (or extract `dist\NetX-v1.0.3-Windows.zip`) to a clean Windows 10/11 test machine (or VM) that does NOT have Python installed.
2. **Demo Mode Test (No Npcap)**:
   - Double-click `NetX.exe`.
   - Verify the browser opens automatically to `http://127.0.0.1:5000`.
   - Verify the warning banner indicates Npcap is missing, but Demo Mode is available.
   - Click **Start Demo**.
   - Verify that simulated traffic, protocol charts, active flows, and DNS queries populate in real-time.
   - Verify that `%LOCALAPPDATA%\NetX\data\netx_demo.db` is created.
   - Click **Stop Demo**.
3. **Live Mode Test (With Npcap)**:
   - Install Npcap from [https://npcap.com](https://npcap.com).
   - Launch `NetX.exe` as Administrator.
   - Verify that Npcap status displays **READY**.
   - Verify the network adapter dropdown automatically lists the clean machine's active network adapter (e.g., Intel Wi-Fi, Realtek Ethernet).
   - Click **Start**.
   - Open another browser tab and browse the web (e.g., stream a video or search).
   - Verify that live packets/sec, bandwidth, and flows reflect real-world traffic.
4. **Shutdown Test**:
   - Click the red **Shutdown** button on the dashboard header.
   - Verify the browser displays the clean shutdown confirmation.
   - Verify that the `NetX.exe` process exits completely in Task Manager without hanging.
5. **Re-launch Test**:
   - Launch `NetX.exe` again.
   - Verify that existing live history buckets and database records reopen seamlessly.

---

## 7. Known Limitations & Defensible Scope

- **Promiscuous Scope**: Captures frames arriving at or originating from the local machine's interface. Whole-LAN visibility requires hardware network taps or SPAN port mirroring.
- **Encrypted Payload Privacy**: Inspects frame headers, 5-tuple flow states, TCP flags, and timing metadata. TLS/HTTPS payloads remain encrypted and are not decrypted.
- **Defensible Assessment**: Anomaly detection identifies behavioral and statistical deviations, attaching mandatory defensible evidence disclaimers to prevent false accusations of malicious activity.
- **High-Speed Saturation**: As a Python/Scapy user-space monitor, sustained packet rates above ~50,000 pps may experience frame drops.
