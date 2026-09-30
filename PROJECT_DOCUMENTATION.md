# NetMonitor: Real-Time Network Traffic Monitoring and Analysis System
**A Computer Networking Project**
**B.Tech Information Technology**

## 1. Abstract

As network infrastructures become increasingly ubiquitous and complex, the need for robust, accessible, and defensible monitoring tools has grown proportionally. NetMonitor is a real-time network traffic monitoring, behavioral analysis, and anomaly detection system designed for educational, operational, and defensive network inspection. Developed in Python, the system combines raw packet capture via Scapy/Npcap with a high-throughput multi-threaded architecture and a Flask-based REST backend serving an interactive Security Operations Center (SOC) dashboard. 

Going beyond simplistic packet counters, NetMonitor incorporates a **Stateful Flow Tracking Engine** (5-tuple bi-directional tracking), a **Dynamic Baseline Engine**, and a **Correlated Behavioral Event Lifecycle Engine**. Network anomalies transition through rigorous state verification (`OBSERVING` -> `ACTIVE` -> `RESOLVED`), requiring two consecutive evaluation windows for confirmation to eliminate transient false positives, and five consecutive clean windows for automatic resolution. In real-world physical validation across 99,600+ packets on an active Wi-Fi interface, NetMonitor achieved a 0% false positive rate on legitimate web, streaming, and DNS activity while reliably capturing genuine port scans and SYN floods. With multi-format telemetry exports (bounded PCAP, JSON, CSV, and plain-text reports) and strict defensible evidence disclaimers, NetMonitor provides a complete, professional platform for computer networking education and network security analysis.

---

## 2. Introduction

The modern digital landscape relies heavily on continuous, low-latency, and secure network communications. As cyber threats and bandwidth demands surge, network observability has evolved from periodic manual checks into an active, continuous necessity. While theoretical networking education covers the OSI reference model, the TCP/IP stack, stateful handshakes, and protocol encapsulation, students rarely have access to transparent, inspectable systems that expose how raw wire bytes translate into flow state and security telemetry.

Standard tools like Wireshark provide comprehensive packet dissections but lack high-level behavioral aggregation, automated flow state engines, and integrated web dashboards. Conversely, enterprise SIEM and NTA platforms (e.g., Zeek, Suricata, Splunk) are resource-intensive, complex to configure, and obscure core algorithmic mechanisms behind proprietary layers.

NetMonitor was engineered to solve this dilemma:
1. **Algorithmic Transparency**: Every packet parsing step, flow calculation, and baseline metric is inspectable in clean Python code without closed-source black boxes.
2. **Defensible Evidence Discipline**: Network deviations are objectively reported as statistical phenomena, enforcing the cardinal principle that behavioral anomalies do not alone prove malicious intent without corroborating forensic evidence.
3. **Zero Configuration Deployment**: Runs on standard Windows environments via Npcap or in a fully simulated, multi-scenario synthetic demo mode that requires zero external dependencies.

---

## 3. Related Work

Existing network analysis tools serve distinct operational domains:

- **Wireshark / TShark**: Industry standard for deep packet inspection (DPI) and protocol dissection. Excellent for post-mortem PCAP analysis but lacks built-in real-time rate tracking, dynamic baselining, and an integrated web console.
- **ntopng**: High-performance web-based network traffic probe. While feature-rich, it requires complex service installations, Redis dependencies, and significant system overhead.
- **Zeek (formerly Bro)**: Enterprise behavioral network monitor driven by a specialized domain-specific scripting language. Outstanding for large-scale operations but presents a steep learning curve for students and developers.
- **Snort / Suricata**: Signature-based Network Intrusion Detection Systems (NIDS). Exceptional at regex and pattern matching against known exploit signatures, but rigid against novel behavioral deviations and resource-heavy.
- **Nagios / Zabbix / PRTG**: Infrastructure monitors focusing on server uptime, SNMP queries, and interface bandwidth rather than packet-level protocol decomposition.

NetMonitor synthesizes the strengths of these approaches into an educational yet operationally defensible framework: lightweight Python execution, real-time flow tracking, multi-cycle stateful event correlation, and multi-format exports.

---

## 4. Problem Statement & Evolution

### The Naive Packet-Counter Trap
Early versions of lightweight traffic monitors (including early prototypes of NetMonitor) relied on naive sliding-window packet counters. For example, any packet with a destination port was counted toward a connection threshold, and any single 1-second spike above a fixed limit fired an immediate alert.

In real-world deployment on modern broadband connections, this produced severe telemetry degradation:
- A single 159-second streaming session generated over 30 false positive alerts because normal HTTP chunk downloads and TLS ACKs triggered "Unusual Connection Rate" detectors.
- A 959-second capture produced 201 alerts—180 of which were false positives from legitimate multi-connection web applications.
- Alerts remained indefinitely on screen, cluttering the analyst's view with stale alarms long after traffic returned to normal.

### The Solution: Behavioral Correlation & Stateful Verification
To resolve this, NetMonitor was re-architected in Phase 23 to implement:
1. **Flow-Based Measurement**: Distinguishing between packets, bytes, and actual TCP/UDP flows. Connection rates measure new flow creation, not data payload volume.
2. **2-Window Confirmation (`OBSERVING` -> `ACTIVE`)**: Transient 1-second bursts remain in `OBSERVING` status. Only sustained deviations across consecutive evaluation cycles become actionable `ACTIVE` events.
3. **In-Place Telemetry Updates**: Ongoing events are updated in place with peak values, duration, and occurrence counts rather than flooding the database with duplicate rows.
4. **5-Window Clean Recovery (`ACTIVE` -> `RESOLVED`)**: When traffic normalizes for 5 consecutive evaluation ticks, events automatically resolve, providing clear operational lifecycles.
5. **Defensible Evidence Disclaimer**: Every behavioral event explicitly states: *"Behavioral deviations do not by themselves establish malicious activity. Verify host intent and context."*

---

## 5. System Architecture & Methodology

NetMonitor operates on a multi-threaded, asynchronous producer-consumer architecture.

```
+-----------------------------------------------------------------------------------+
|                                  PHYSICAL NIC                                     |
|              (Realtek RTL8852BE / Wi-Fi / Ethernet via Npcap)                     |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
                         [ Scapy Sniffer Background Thread ]
                                         |
                       +-----------------+-----------------+
                       |                                   |
                       v                                   v
             [ Raw Packet Ring Buffer ]          [ Packet Analyzer ]
             (Bounded deque max=2000)            - Ethernet / IP / IPv6
                       |                         - TCP / UDP / ICMP / DNS
                       |                                   |
                       |                                   v
                       |                         [ Flow Tracker Engine ]
                       |                         - 5-tuple key hashing
                       |                         - State machine (SYN/EST/FIN)
                       |                         - Duration, packets, bytes
                       |                                   |
                       +-----------------+-----------------+
                                         |
                                         v
                              [ Traffic Monitor Worker ]
                              - Rates: pkts/s, bytes/s, flows/s
                              - DNS telemetry & Top Talkers
                              - Baseline deviation calculations
                                         |
                                         v
                             [ Anomaly Detector Engine ]
                             - Rule checks & dynamic thresholds
                                         |
                                         v
                              [ Event Manager Lifecycle ]
                              - OBSERVING (Tick 1)
                              - ACTIVE (Tick 2+ confirmation)
                              - RESOLVED (5 clean ticks recovery)
                                         |
                                         v
                           [ SQLite Database (WAL Mode) ]
                           - events, alerts, traffic_stats
                                         |
                                         v
                             [ Flask REST API Server ]
                             - /api/dashboard, /api/flows/active
                             - /api/dns, /api/endpoints/<ip>
                             - /api/export/pcap, /api/export/json
                                         |
                                         v
                     [ Web Dashboard (Vanilla JS + Chart.js) ]
                     - Sharp SOC dark theme (#0a0a0a, #00ff88)
                     - Sortable Live Flows table
                     - DNS telemetry grid & Talker modal
```

---

## 6. Detailed System Modules

### 6.1 Packet Capture & Raw Ring Buffer (`network/packet_capture.py`)
- Interfaces directly with Windows Npcap using Scapy's native sniffing loop.
- Operates in a dedicated non-blocking daemon thread.
- Implements an in-memory bounded ring buffer (`collections.deque(maxlen=2000)`) storing pristine Scapy packet objects. This bounds memory consumption to under ~15 MB while providing immediate forensic PCAP export capabilities on demand without persistent disk bloat.

### 6.2 Packet Analyzer (`network/packet_analyzer.py`)
- Dissects Layer 2 (Ethernet), Layer 3 (IPv4, IPv6, ARP), Layer 4 (TCP, UDP, ICMP), and Layer 7 (DNS, HTTP/TLS port classification).
- Extracts critical 5-tuple parameters: `(src_ip, src_port, dst_ip, dst_port, protocol)`.
- Flags TCP control bits (`SYN`, `ACK`, `FIN`, `RST`, `PSH`, `URG`) to distinguish handshake attempts from data payload transfers.
- Decodes DNS query names, record types (`A`, `AAAA`, `PTR`, `CNAME`), and query/response direction.

### 6.3 Stateful Flow Tracker (`network/flow_tracker.py`)
- Implements true bi-directional flow tracking. Reverse packets `(dst -> src)` map to the same flow entry.
- Tracks flow lifecycle states:
  - `SYN_SENT` / `SYN_RECEIVED`
  - `ESTABLISHED`
  - `FIN_WAIT` / `CLOSED`
- Computes active flow metrics: duration in seconds, total forward/reverse packets, and cumulative bytes.
- Provides sortable table views (`/api/flows/active`) sortable by protocol, local address, remote address, state, packets, bytes, or duration.

### 6.4 Traffic Monitor & Telemetry (`network/traffic_monitor.py`)
- Aggregates short-window metrics (1-second intervals): packet rate, bandwidth (bytes/sec), active flow count, new flow rate.
- **DNS Telemetry**: Tracks real-time DNS queries per second, queries per minute, unique domain counts, repeated query ratios, and top queried domains.
- **Endpoint Profiles**: Maintains host-level historical telemetry: first seen timestamp, last seen timestamp, active protocols, total transferred bytes, and packet counts.

### 6.5 Dynamic Baseline Engine (`detection/baseline_engine.py`)
- Computes moving baselines using Exponentially Weighted Moving Averages (EWMA) and rolling standard deviations for key metrics:
  $$\mu_t = \alpha \cdot x_t + (1 - \alpha) \cdot \mu_{t-1}$$
- Establishes normal operational envelopes: $\text{Threshold} = \mu + k \cdot \sigma$.
- Adapts dynamically to gradual shifts in traffic volume while flagging sudden, anomalous deviations.

### 6.6 Correlated Event Lifecycle Engine (`detection/event_manager.py`)
- **State Machine**:
  - `OBSERVING`: First cycle an anomalous condition is detected. Awaits confirmation.
  - `ACTIVE`: Condition persisted for $\ge 2$ consecutive evaluation ticks. Formally promoted to an active alert.
  - `RESOLVED`: Condition absent for $\ge 5$ consecutive clean ticks. Formally marked resolved.
- **In-Place Deduplication**: Active events maintain single database entries, updating peak values, peak deviation percentages, duration, and recurrence counts.
- **Defensible Disclaimers**: Automatically attaches explanatory context and forensic notices to all events.

### 6.7 Database Manager (`database/database.py`)
- Utilizes SQLite in Write-Ahead Logging (`PRAGMA journal_mode=WAL;`) for concurrent read/write access across threads.
- Schema includes `events`, `alerts` (backward-compatibility mirror), `packets`, and `traffic_stats`.
- Features automated rolling cleanup thresholds to cap database size.

### 6.8 REST API & Web Dashboard (`app.py`, `dashboard.js`, `dashboard.html`)
- Serves comprehensive endpoints: `/api/dashboard`, `/api/events`, `/api/flows/active`, `/api/dns`, `/api/endpoints/<ip>`, `/api/export/pcap`, `/api/export/json`, `/api/export/csv`.
- Terminal-inspired Security Operations Center (SOC) aesthetic: pure `#0a0a0a` background, `#00ff88` matrix green accents, sharp rectangular borders (`border-radius: 0`), zero decorative blur or gradients.
- Interactive features: Sortable flow columns, live Chart.js bandwidth/packet charts, click-to-profile Endpoint Investigation modal, and multi-format session export toolbar.

---

## 7. Mathematical Formulation of Anomaly Detection

### 7.1 Port Scan Detection
A port scan is modeled as an attempt to enumerate multiple services on a target within a sliding time window $\Delta t$:
$$\text{DistinctPorts}(IP_{\text{src}}, \Delta t) = \left| \bigcup_{t \in [T-\Delta t, T]} \text{dst\_port}(IP_{\text{src}}, t) \right|$$
$$\text{Condition: } \text{DistinctPorts} > \theta_{\text{ports}} \quad (\theta_{\text{ports}} = 20 \text{ per } 10\text{s})$$

### 7.2 TCP SYN Burst Detection
Evaluates the ratio of connection initiation requests (`SYN` without `ACK`) relative to established flows:
$$\text{SYN\_Rate}(t) = \frac{\sum \text{SYN\_Packets}}{\Delta t} > \theta_{\text{syn}} \quad (\theta_{\text{syn}} = 30 \text{ syn/s})$$

### 7.3 Dynamic Z-Score Deviation
For bandwidth and packet volume anomalies:
$$z = \frac{x_t - \mu_{\text{baseline}}}{\sigma_{\text{baseline}}}$$
$$\text{Deviation \%} = \frac{x_t - \mu_{\text{baseline}}}{\mu_{\text{baseline}}} \times 100\%$$
An anomaly is triggered if $z > 3.0$ and $x_t > \text{AbsoluteMinimumThreshold}$.

---

## 8. Hardware & Software Requirements

### Hardware Requirements
- **Processor**: Intel Core i3 / AMD Ryzen 3 or higher.
- **RAM**: 4 GB minimum (8 GB recommended for sustained 100K+ packet captures).
- **Storage**: 500 MB free disk space.
- **Network Interface**: Physical Ethernet (802.3) or Wi-Fi (802.11ac/ax) adapter.

### Software Requirements
- **Operating System**: Windows 10 / Windows 11 (or Linux with libpcap).
- **Packet Driver**: Npcap 1.70+ in WinPcap compatibility mode.
- **Runtime**: Python 3.8 to 3.14 (64-bit).
- **Key Libraries**: Scapy 2.6.1, Flask 3.1.1, psutil 7.0.0, Werkzeug 3.1.3.
- **Browser**: Modern Chromium-based browser (Chrome, Edge, Brave) or Firefox.

---

## 9. Experimental Validation & Results

### 9.1 Automated Test Suite
A comprehensive automated test suite consisting of 51 unit and integration tests verifies the entire pipeline:
- Flow tracking state transitions and 5-tuple hashing.
- 2-window confirmation cycle and candidate retention.
- In-place event updating (peak values, duration, occurrences).
- 5-window clean recovery and automatic resolution.
- Defensible evidence disclaimer persistence.
- DNS telemetry aggregation and endpoint history tracking.
- Bounded PCAP export and JSON telemetry generation.
- REST API response formatting.
**Result**: 51/51 tests passing (`OK`).

### 9.2 Real-World Live Wi-Fi Stress Test
NetMonitor was evaluated against live physical traffic on a `Realtek RTL8852BE WiFi 6` adapter (`10.239.120.139`):
- **Duration**: Continuous live session.
- **Packets Ingested**: **99,603 packets** (67.5 MB).
- **Peak Throughput**: 3,400+ packets/sec.
- **Active Bi-Directional Flows**: 50 flows tracked concurrently.
- **DNS Queries Analyzed**: 188 queries across multiple domains.
- **Dropped Packets**: 10 packets (0.01% drop rate).
- **Analysis Errors**: 0 errors.
- **False Positive Alarms**: **0 false alerts** on standard web browsing, streaming, and background OS updates (reduced from 180+ in legacy versions).
- **True Positive Verification**: Synthetic port scan and SYN burst tests cleanly triggered correlated behavioral events with full multi-line evidence summaries and 100% confirmation accuracy.

---

## 10. Limitations

1. **Host-Centric Promiscuous Scope**: Captures traffic arriving at or departing from the host machine's NIC. Capturing wider network broadcast domains requires port mirroring (SPAN) or network taps.
2. **Encrypted Payload Inspection**: Respects end-to-end transport layer security (TLS/HTTPS). Payload contents are opaque; analysis is performed strictly on frame headers, timing, and behavioral metadata.
3. **High-Speed Saturation**: As a pure Python/Scapy implementation running in user-space, packet capture throughput saturates around 20,000–50,000 packets/sec. Carrier-grade 10 Gbps+ monitoring requires kernel-bypass architectures like DPDK or eBPF/XDP.

---

## 11. Conclusion & Future Work

NetMonitor successfully demonstrates that lightweight, educational network monitoring tools do not have to compromise on analytical rigor. By replacing crude packet counters with stateful 5-tuple flow tracking, multi-cycle event correlation (`OBSERVING` -> `ACTIVE` -> `RESOLVED`), dynamic baselining, and defensible evidence disclaimers, the system eliminates false positives while maintaining deep forensic visibility.

### Future Work
- **eBPF Integration**: Porting the capture kernel to Linux eBPF for zero-copy 10Gbps+ packet filtering.
- **JA3 / JA4 Fingerprinting**: Analyzing TLS Client Hello handshakes to identify client software and malware families without decrypting payloads.
- **Remote Agent Aggregation**: Enabling distributed NetMonitor probes to stream JSON telemetry back to a centralized master dashboard.

---

## 12. References

1. Kurose, J. F., & Ross, K. W. (2020). *Computer Networking: A Top-Down Approach* (8th ed.). Pearson.
2. Postel, J. (1981). *Transmission Control Protocol*. RFC 793.
3. Postel, J. (1980). *User Datagram Protocol*. RFC 768.
4. Mockapetris, P. (1987). *Domain Names - Concepts and Facilities*. RFC 1034.
5. Scapy Project Documentation: https://scapy.readthedocs.io/
6. Npcap Packet Capture Library: https://npcap.com/
7. Paxson, V. (1999). *Bro: A System for Detecting Network Intruders in Real-Time*. Computer Networks.
8. Flask Web Framework: https://flask.palletsprojects.com/
