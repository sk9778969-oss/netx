================================================================================
                    NETX - NETWORK MONITOR & ANALYZER
================================================================================

NetX is a standalone, real-time network traffic analysis and monitoring
application for Windows. It provides live packet capture via Npcap, traffic 
analytics, top talkers, connection tracking, explainable anomaly heuristics, and
data export capabilities.

================================================================================
1. HOW TO START NETX
================================================================================

1. Double-click "NetX.exe".
2. The application will start its local backend service.
3. Your default web browser will automatically open to:
   http://127.0.0.1:5000/
4. You do NOT need Python, pip, or a command prompt installed to run this app.

================================================================================
2. HOW TO SELECT THE NETWORK ADAPTER
================================================================================

- Upon opening, NetX automatically detects and selects your primary
  active Internet network adapter (e.g., Wi-Fi or Ethernet).
- You can manually choose a different adapter from the dropdown list located
  in the top header bar.
- Disconnected adapters, loopback interfaces, Bluetooth links, and virtual
  adapters (e.g. Wi-Fi Direct) are automatically deprioritized.

================================================================================
3. HOW TO USE LIVE MODE
================================================================================

1. Verify that your active network adapter is selected in the dropdown.
2. Click the "START" button in the top navigation bar.
3. The "LIVE" indicator will appear in green.
4. Packets captured from your real network interface will immediately populate
   the dashboard:
   - Live Traffic graph (Packets/sec and Bandwidth KB/s)
   - Protocol Distribution (TCP, UDP, DNS, ICMP, OTHER)
   - Top Talkers (IP addresses generating the most volume)
   - Top Connections (source and destination sockets)
   - Recent Packets table (newest packets with ports and sizes)
5. You can PAUSE capture at any time and RESUME when ready.

================================================================================
4. HOW TO USE DEMO MODE
================================================================================

- If Npcap is not installed or you wish to test the interface without capturing
  real network traffic, click the "START DEMO" button.
- NetX will generate simulated packets through the exact same analytics,
  database, and anomaly detection pipeline.
- An amber banner will clearly indicate that simulated demo data is active.
- Demo mode periodically tests security heuristics (port scans, SYN bursts,
  ICMP floods, DNS bursts) to demonstrate explainable alert capabilities.
- Click "STOP DEMO" to exit demo mode.

================================================================================
5. NPCAP REQUIREMENT
================================================================================

- LIVE mode requires Npcap (or WinPcap) to capture raw network packets from
  Windows network adapters.
- If Npcap is missing, LIVE mode will alert you and suggest using DEMO mode.
- See "Npcap_Installation.txt" in this package for complete step-by-step
  installation instructions.

================================================================================
6. FILTERING & CSV EXPORT
================================================================================

- TIME FILTER: Use the Time Filter dropdown to query packets from "All Time",
  "Last 1 min", "Last 5 min", or "Last 15 min".
- SEARCH & PROTOCOL: Filter packets by specific protocol (TCP, UDP, DNS, ICMP)
  or search by specific IP address.
- EXPORT PACKETS: Click "Export Pkts" to download a clean CSV file of captured
  packets (timestamp, source, destination, protocol, ports, size).
- EXPORT ALERTS: Click "Export Events" to download a CSV file of all detected
  anomalies with explainable detection reasons and thresholds.
- NetX never exports raw payloads or sensitive credentials.

================================================================================
7. TROUBLESHOOTING
================================================================================

Q: The browser didn't open automatically.
A: Open any browser and manually navigate to: http://127.0.0.1:5000/

Q: "Npcap or WinPcap is required for capturing packets" error appears.
A: Install Npcap from https://npcap.com/ and make sure to check "Install Npcap
   in WinPcap API-compatible Mode" during setup.

Q: No packets appear when clicking START in LIVE mode.
A: Ensure you have selected the adapter that has active Internet traffic.
   Check the "Network Overview" bar on the dashboard: Local IP and Gateway
   should be visible, and Link Status should show "CONNECTED".

Q: Port 5000 is already in use.
A: Close any other running instances of NetX or other local servers
   occupying port 5000 via Windows Task Manager.

================================================================================
8. HOW TO STOP THE APPLICATION
================================================================================

- To stop capturing: Click "STOP" on the dashboard.
- To terminate NetX completely: Click "Shutdown" in the top bar, or close the
  browser window and end the "NetX.exe" process from Windows Task Manager.
================================================================================
