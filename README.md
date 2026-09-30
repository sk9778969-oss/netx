# NetX

**Defensible Real-Time Network Traffic Monitoring & Behavioral Analysis System**

NetX (formerly NetMonitor) is a high-performance, Python-based network traffic monitoring and behavioral telemetry engine. Designed for educational inspection, network operations, and defensive security analysis, NetX combines Scapy/Npcap packet capture with stateful 5-tuple flow tracking, dynamic baselining, correlated event lifecycles, and a Security Operations Center (SOC) style web console.

---

## Key Features

- **Live & Deterministic Demo Capture**:
  - **Live Mode**: Real-time packet sniffing directly from physical interfaces (Wi-Fi, Ethernet) via Windows Npcap.
  - **Demo Mode**: Reproducible scenario simulations (`normal`, `port_scan`, `syn_burst`, `high_traffic`, `dns_burst`, `video_stream`).
- **Stateful Bi-Directional Flow Tracking**:
  - 5-tuple tracking `(src_ip, src_port, dst_ip, dst_port, protocol)` with TCP handshake state tracking (`SYN_SENT`, `ESTABLISHED`, `FIN_WAIT`).
  - Interactive Live Flow table sortable by Protocol, Local Address, Remote Address, State, Packets, Bytes, and Duration.
- **Correlated Behavioral Traffic Events**:
  - **Lifecycle Engine**: Events transition through `OBSERVING` -> `ACTIVE` -> `RESOLVED`.
  - **Confirmation Window (2 cycles)**: Transient 1-second bursts are evaluated in observation mode; only sustained deviations become active alerts, eliminating transient false positives.
  - **In-Place Updates**: Active events are updated in place with peak rates, peak deviation %, duration, and recurrence counts without database flooding.
  - **Clean Recovery Window (5 cycles)**: Events automatically resolve after 5 consecutive clean evaluation cycles.
- **Defensible Evidence Discipline**:
  - Every alert and behavioral event includes a mandatory defensible context notice: *"Behavioral deviations do not by themselves establish malicious activity. Verify host intent and context."*
- **Real-Time DNS Analytics**:
  - Tracks DNS queries/sec, queries/min, unique domains, repeated query ratios, and top queried domains.
- **Endpoint Profiling & Top Talkers**:
  - Host-level drill-down modal showing first seen, last seen, active protocols, transferred bytes, and traffic share percentage.
- **Multi-Format Session Exports**:
  - **PCAP Export**: Non-destructive, bounded raw packet buffer export (`.pcap`) for Wireshark inspection.
  - **JSON Telemetry**: Machine-readable full system snapshot (`/api/export/json`).
  - **CSV Exports**: Filtered packet logs (`/api/export/packets`) and event logs (`/api/export/alerts`).
  - **Plain-Text Report**: Formatted forensic investigation summary (`/api/export/report`).
- **Sharp SOC Dashboard**:
  - Black `#0a0a0a` background, `#00ff88` matrix green accents, sharp rectangular borders (`border-radius: 0`), zero gradients, and real-time Chart.js graphs.

---

## Prerequisites & External Dependencies

- **Operating System**: Windows 10 or 11 (64-bit)
- **Python**: 3.8+ (Required only when running from source; not required when using `NetX.exe`)
- **Npcap Driver** (Required for LIVE capture):
  - Live packet sniffing on Windows requires the official [Npcap Packet Driver](https://npcap.com/).
  - **Important**: Npcap is an external kernel-mode network driver, **not** a Python package that PyInstaller or pip can install automatically.
  - During Npcap setup, make sure **"Install Npcap in WinPcap API-Compatible Mode"** is enabled.
  - *Administrator Privileges*: Windows requires Administrator privileges during driver installation. For day-to-day monitoring, standard privileges suffice unless the driver was installed with *"Restrict Npcap driver's access to Administrators only"*, in which case NetX must be launched with **Run as administrator**.
- **DEMO Mode**:
  - Requires **neither** Npcap nor Administrator privileges. Users can explore all synthetic traffic scenarios, flow analysis, and detection baselines immediately on any system.

---

## Installation & First-Run Setup Experience

### Option 1 — Standalone Windows Executable (Recommended)

1. Launch `NetX.exe` (from the distribution release archive `NetX-v1.0.3-Windows.zip` or compiled `dist\NetX.exe`).
2. NetX performs non-blocking startup checks, allocates a free local port, and automatically opens the dashboard in your default browser (`http://127.0.0.1:5000`).
3. **If Npcap is not detected**:
   - The dashboard opens gracefully without crashing.
   - A first-run setup dialog appears explaining that LIVE capture is unavailable.
   - Click the **Install Npcap** button to open the official download page: [https://npcap.com/#download](https://npcap.com/#download).
   - Follow the concise setup steps (ensure WinPcap API-compatible mode is enabled).
   - Once installation finishes, click **Recheck Status** directly on the dashboard &mdash; NetX immediately detects the driver without requiring an application restart or reinstall!
   - Alternatively, click **Start Demo Mode** to explore the application immediately without installing drivers.
4. When Npcap is detected, select your active physical network adapter (Wi-Fi or Ethernet) and click **Start** to begin live packet capture.

### Option 2 — Run From Source

```bash
# Set up virtual environment
python -m venv .venv
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run backend server
python app.py
```
Open your browser and navigate to: [http://127.0.0.1:5000](http://127.0.0.1:5000)

---

## Modes of Operation

- **LIVE Mode**: Captures live packets from your selected physical adapter (e.g., `Wi-Fi`, `Ethernet`, or USB network interface). Requires Npcap installed with WinPcap compatibility and Administrator privileges for raw socket access. Real-world traffic telemetry is persistently stored in `%LOCALAPPDATA%\NetX\data\netx.db` (with automatic fallback/preservation of legacy `%LOCALAPPDATA%\NetMonitor\data\netmonitor.db`).
- **DEMO Mode**: Synthesizes deterministic, mathematically consistent network traffic scenarios. Does **NOT** require Npcap or Administrator privileges. Simulation data is physically isolated in `netx_demo.db` and never contaminates real network history.

---

## REST API Reference

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/api/dashboard` | `GET` | Aggregated dashboard telemetry (rates, protocol distribution, top talkers, baselines) |
| `/api/events` | `GET` | Correlated behavioral events with active/observing/resolved status |
| `/api/events/<event_id>` | `GET` | Detailed telemetry and defensible evidence for a specific event |
| `/api/flows/active` | `GET` | Active bi-directional flows with sorting (`sort_by`, `order`, `limit`) |
| `/api/dns` | `GET` | Real-time DNS metrics, rates, and top queried domains |
| `/api/endpoints/<ip>` | `GET` | Endpoint profile (first/last seen, protocols, packets, bytes) |
| `/api/export/pcap` | `GET` | Forensic raw `.pcap` capture file download |
| `/api/export/json` | `GET` | Complete machine-readable session state JSON snapshot |
| `/api/export/csv` | `GET` | Packet log CSV export |
| `/api/export/events/csv` | `GET` | Behavioral events CSV export |
| `/api/export/report` | `GET` | Formatted plain-text network traffic report |
| `/api/demo/start` | `POST` | Start synthetic demo scenario (`{"scenario": "port_scan"}`) |

---

## Project Structure

```
NetMonitor/
├── app.py                      # Main entry point & Flask REST API
├── config.py                   # Central configuration & detection thresholds
├── requirements.txt            # Python dependencies
├── README.md                   # Project overview & documentation
├── PROJECT_DOCUMENTATION.md    # Academic technical specification
├── VIVA.md                     # Comprehensive viva Q&A guide
├── BUILD.md                    # PyInstaller build instructions
├── netmonitor.spec             # PyInstaller packaging spec
├── network/
│   ├── interface_manager.py    # NIC enumeration via psutil
│   ├── packet_capture.py       # Scapy sniffer & bounded raw packet ring buffer
│   ├── packet_analyzer.py      # Layer 2-7 packet dissector & header parsing
│   ├── flow_tracker.py         # 5-tuple bi-directional stateful flow tracker
│   └── traffic_monitor.py      # Rates, DNS telemetry & endpoint profiling
├── detection/
│   ├── anomaly_detector.py     # Rule-based detection & threshold logic
│   ├── baseline_engine.py      # EWMA moving baselines & z-score deviations
│   └── event_manager.py        # Correlated behavioral lifecycle engine (OBSERVING/ACTIVE/RESOLVED)
├── database/
│   └── database.py             # SQLite WAL persistence, events table & auto-cleanup
├── demo/
│   └── demo_generator.py       # Deterministic scenario generator & synthetic PCAP
├── templates/
│   └── dashboard.html          # High-density SOC dashboard template
├── static/
│   ├── css/
│   │   └── style.css           # Terminal dark theme (#0a0a0a, #00ff88, rectangular)
│   └── js/
│       └── dashboard.js        # Polling, Chart.js graphs, sortable flows & modals
├── tests/                      # Automated test suite (51 tests)
└── dist/
    └── NetMonitor.exe          # Standalone Windows executable
```

---

## Testing

Run the automated test suite:
```bash
python -m unittest discover -s tests -p "test_*.py"
```

---

## Executable Build Process

To compile NetX into a standalone, single-file Windows executable (`NetX.exe`) using PyInstaller:

```bash
# Using the automated build script (runs tests & packages release ZIP)
python build.py

# Or using the provided spec file
pyinstaller --noconfirm netx.spec
```
For detailed compilation notes, troubleshooting, and packaging options, see [`BUILD.md`](BUILD.md).

---

## Limitations & Defensible Scope

- **Promiscuous Scope**: Captures frames arriving at or originating from the local host's network interfaces. Monitoring whole-LAN broadcast domains requires network taps or port mirroring (SPAN).
- **Transport Security (TLS/HTTPS)**: Analyzes frame headers, 5-tuple flow states, and temporal behavior. Payloads encrypted with TLS remain confidential and are not decrypted.
- **Defensible Behavioral Assessment**: Anomaly metrics (e.g., Z-scores, connection rates) quantify statistical deviations rather than definitive malicious intent. Automated events include disclaimers reminding analysts to corroborate host context and application behavior before reaching operational conclusions.
- **Capture Throughput**: Designed as an educational and defensive Python/Scapy tool running in user-space. Environments with sustained >50,000 pps throughput require kernel-bypass architectures (e.g., DPDK, eBPF).

---

## Repository Maintenance & Contributions

- **Official Repository**: This repository (`sk9778969-oss/NetX`) is the official upstream codebase maintained exclusively by its owner.
- **Contributions**: External contributors are requested to open an issue and obtain maintainer approval before submitting code or pull requests. Unsolicited pull requests are not automatically accepted.
- **Contribution Guidelines**: Please review [`CONTRIBUTING.md`](CONTRIBUTING.md) for contribution workflows, code quality requirements, and review criteria.

---

## License & Copyright

Copyright (c) 2026 Vasireddy Sanjay Kumar. All rights reserved.

NetX (formerly NetMonitor) is provided under an **All Rights Reserved** notice for personal, educational, and evaluation purposes. Public redistribution, modification, or commercial exploitation is prohibited without prior written consent. See [`LICENSE`](LICENSE) for terms.
