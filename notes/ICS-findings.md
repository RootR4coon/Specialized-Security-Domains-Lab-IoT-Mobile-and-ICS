# ICS Findings

Detailed methodology and evidence: [docs/03-ics.md](../docs/03-ics.md).

| ID | Title | Severity | Key evidence | Recommendation |
|---|---|---|---|---|
| ICS-01 | Docker Desktop for macOS fails to enforce loopback-only binding on port-remapped publishes; ports 102 and 502 exposed on all interfaces despite `127.0.0.1` binding | High | `28-ics-exposure-test.png`, `27-ics-lsof-interfaces.png` | Do not rely on remapped loopback publishes as a security boundary; match port numbers, add host firewall rules, or use native Linux Docker |
| ICS-02 | Modbus has no authentication or encryption; unauthenticated write command (function code 6) processed at protocol level | High | `26-ics-wireshark-modbus-write.png`, `27-ics-wireshark-write-filter.png` | Segment per IEC 62443; authenticate at a gateway; IDS signature for write function codes 5/6/15/16 |

## NIST SP 800-82 mini assessment

| Area | Observation | Risk | Recommendation |
|---|---|---|---|
| Authentication | Modbus has none; unauthenticated write processed | High | Segment; gateways that add authentication |
| Encryption | All traffic cleartext | High | VPN or secure tunnels between zones |
| Exposure | Remapped ports reachable externally despite loopback binding | High | Independent external verification; host firewall second layer |
| Monitoring | No alerting by default | High | IDS/IPS on tap/SPAN port with write-code signature |

## IEC 62443 zones and conduits

| Zone | Devices | Allowed Communication (Conduit) |
|---|---|---|
| Enterprise (IT) | Office PCs, email | Only to the DMZ |
| DMZ | Historian, remote access gateway | To the control zone, on specific ports only |
| Control | PLCs, HMIs | Only from the engineering workstation and HMI |
| Safety | Safety controllers | Isolated; no inbound traffic from IT |
