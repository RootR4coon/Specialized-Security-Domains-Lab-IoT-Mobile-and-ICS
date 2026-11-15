# 3. Industrial Control Systems (ICS) Security

Back to [README](../README.md). Findings table: [notes/ics-findings.md](../notes/ics-findings.md).

## 3.1 Lab Environment (Conpot)

Started Conpot, a simulated Siemens PLC honeypot, in Docker:

```bash
docker run -d --name conpot \
  -p 127.0.0.1:8800:8800 \
  -p 127.0.0.1:102:10201 \
  -p 127.0.0.1:502:5020 \
  honeynet/conpot
```

Confirmed running via `docker ps` and `docker logs conpot`, which showed the Modbus server started on `0.0.0.0:5020` (inside the container), the S7Comm server on `0.0.0.0:10201`, and the HTTP server on `0.0.0.0:8800`. The web interface at `http://localhost:8800` displayed the simulated PLC page ("Technodrome," Conpot's default S7-300-style template).

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/3428b45a-8264-4324-a738-82a368c5d6f0"
      <br><sub>Conpot Container</sub>
    </td>
  </tr>
</table>

## 3.2 Discovery and Analysis

**Nmap service scan:**

```bash
nmap -sV -p 102,502,8800 127.0.0.1
```

| Port | State | Service (Nmap identification) |
|---|---|---|
| 102/tcp | open | `iso-tsap?` (S7comm transport) |
| 502/tcp | open | `mbap?` (Modbus Application Protocol) |
| 8800/tcp | open | Misidentified as `sunwebadmin` (Nmap's fingerprint database has no entry for Conpot's simulated HTTP interface) |

**NSE identity scripts:**

```bash
nmap -p 502 --script modbus-discover 127.0.0.1
nmap -p 102 --script s7-info 127.0.0.1
```

Both scripts returned no identification data. Conpot's emulation is accurate enough to answer basic service and version probes, but does not implement the deeper identity-query responses these scripts expect from genuine hardware.

**Device type and vendor summary:** Simulated PLC honeypot configured to emulate a Siemens S7-300-series controller, based on the S7comm transport on port 102, the "Technodrome" HTTP template, and Docker logs explicitly naming the S7Comm server.

PLCscan (optional, Python 2) was not used; the lab guide confirms the Nmap scripts above provide equivalent information.

## 3.3 Protocol Traffic Analysis (Wireshark)

Captured on the loopback interface with filter `mbtcp || s7comm`.

**Read operation:**

```bash
mbpoll -1 -a 1 -r 1 -c 5 -t 4 -p 502 127.0.0.1
```

Generated a function code 4 (Read Input Registers) request. The response returned a Modbus exception, "Illegal Data Address," because Conpot's simulated register map has no data at that address. This is a valid protocol-level response, not a failure of the test.

**Write operation:**

```bash
mbpoll -1 -a 1 -r 1 -t 4 -p 502 127.0.0.1 123
```

Generated a function code 6 (Write Single Register) request, processed with no authentication or credentials of any kind; the value `123` (`0x7b`) was sent in plain text. The response was again an exception due to the invalid address, but the write attempt itself was accepted and processed at the protocol layer with no access control whatsoever.

**Write-detection filter built and tested:**

```
modbus.func_code == 5 || modbus.func_code == 6 || modbus.func_code == 15 || modbus.func_code == 16
```

**Function code reference, from traffic observed in this lab:**

| Function Code | Operation | Type | Observed |
|---|---|---|---|
| 3 | Read Holding Registers | Read | Yes |
| 4 | Read Input Registers | Read | Yes |
| 5 | Write Single Coil | Write | Not generated |
| 6 | Write Single Register | Write | Yes |
| 15 | Write Multiple Coils | Write | Not generated |
| 16 | Write Multiple Registers | Write | Not generated |

**Key discovery, confirmed firsthand:** Modbus has no authentication and no encryption. A write command was accepted with no credentials; the only thing preventing a successful write was Conpot's own internal memory-map limitation, not any access control mechanism.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ccc59fd7-c323-47ad-8986-ffb6de3386e5"
      <br><sub>modus TCP</sub>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/7f78d26a-715d-4e7b-91a2-73b63ccdbd7f"
      <br><sub>modus TCP</sub>
   <td align="center">
      <img src="https://github.com/user-attachments/assets/5ffa9b82-8b5b-441c-b9dd-4d9e1ab51e96"
      <br><sub>modus command Wireshark</sub>
    </td>
  </tr>
</table>

## 3.4 Security Controls

### Control 1: Network Exposure Test

Verified Conpot's port bindings from a separate machine (Kali) on the same network, targeting the host machine's LAN IP.

| Port | Expected | Actual Result from Kali |
|---|---|---|
| 8800 (`8800:8800`, matching port numbers) | Closed or filtered | Closed, as expected |
| 102 (`102:10201`, remapped) | Closed or filtered | Open, unexpected |
| 502 (`502:5020`, remapped) | Closed or filtered | Open, unexpected |

**Root cause, confirmed via `lsof -i :<port>` on the host:**

```text
Port 102:  *:iso-tsap        (LISTEN) -- all interfaces
Port 502:  *:asa-appl-proto  (LISTEN) -- all interfaces
Port 8800: localhost:sunwebadmin (LISTEN) -- loopback only, correct
```

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/0e9d4ea5-75ac-4c23-96e7-6dba483a1cc9"
      <br><sub>Silently broadens all * interfaces</sub>
    </td>
  </tr>
</table>

### Finding ICS-01: Docker Desktop for macOS Fails to Enforce Loopback-Only Binding on Port-Remapped Publishes

- **Detail:** `docker run -p 127.0.0.1:host:container` was used for all three ports. `docker port conpot` correctly reported all three as bound to `127.0.0.1`. However, only the port where the host and container numbers matched (`8800:8800`) was actually restricted to loopback at the operating-system level. Both remapped ports (`102:10201`, `502:5020`) were listening on all interfaces, with no error, warning, or difference in `docker ps` or `docker port` output to indicate this.
- **Impact:** A security control believed to be in place, network isolation via loopback binding, was silently absent for two of three published ports. This would not have been discovered without independent external verification.
- **Severity:** High
- **Recommendation:** Do not rely on `-p 127.0.0.1:host:container` alone as a security boundary on Docker Desktop for macOS when host and container port numbers differ. Match port numbers where possible, add an independent host firewall rule as a second layer of defense, or run this type of exercise on native Linux Docker, where this inconsistency was not observed.
- **Reference:** Docker Desktop macOS networking architecture (VM and proxy-based); general defense-in-depth principle.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/b38ed83e-c880-4504-818e-4c049bd9d154"
      <br><sub>Nmap scan</sub>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/240f6422-8995-454b-848f-94ad74d696ab"
      <br><sub>The Port is Open</sub>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/8dfaab2b-1929-44b1-9f61-4ade10c7399e"
      <br><sub>Port 502 open</sub>
    </td>
  </tr>
</table>

### Control 2: Segmentation Plan (IEC 62443 Zones and Conduits)

| Zone | Devices | Allowed Communication (Conduit) |
|---|---|---|
| Enterprise (IT) | Office PCs, email | Only to the DMZ |
| DMZ | Historian, remote access gateway | To the control zone, on specific ports only |
| Control | PLCs, HMIs | Only from the engineering workstation and HMI |
| Safety | Safety controllers | Isolated; no inbound traffic from IT |

### Control 3: Monitoring

Saved filter (identical to Section 3.3):

```
modbus.func_code == 5 || modbus.func_code == 6 || modbus.func_code == 15 || modbus.func_code == 16
```

**Production alerting approach:** A network tap or SPAN port feeding an intrusion detection system, such as Suricata or Zeek, could apply this same filter logic as a signature, triggering an alert whenever a Modbus write function code (5, 6, 15, or 16) appears on the wire. This flags every write attempt for review, since on an unauthenticated protocol such as Modbus, any write command from an unexpected source is inherently suspicious.

### Control 4: Hardening Checklist

- [x] Change default passwords on all devices
- [x] Disable unused services and ports. Conpot was observed running SNMP, Bacnet, IPMI, FTP, and TFTP beyond the scope of this test, exactly the kind of unnecessary exposure this control addresses.
- [x] Restrict remote access to a VPN with multi-factor authentication. Directly motivated by the confirmed lack of Modbus authentication.
- [x] Keep an up-to-date asset inventory
- [x] Back up PLC programs and configurations offline
- [x] Patch with a tested, scheduled process

## 3.5 Mini ICS Security Assessment (NIST SP 800-82)

| Area | Observation from the Lab | Risk | Recommendation |
|---|---|---|---|
| Authentication | Modbus has none. Confirmed firsthand by sending an unauthenticated write command (function code 6), which was processed at the protocol level. | High | Segment the network; use gateways or proxies that add an authentication layer in front of the protocol itself. |
| Encryption | All traffic observed in Wireshark was plain text; function codes, addresses, and values were all readable in plain hex. | High | Use a VPN or secure tunnel (IPsec, stunnel) between zones. Modbus/TCP has no native encryption. |
| Exposure | Ports reachable from another machine despite explicit `127.0.0.1`-only binding on remapped ports 102 and 502; the same-numbered port 8800 was correctly restricted. See Finding ICS-01. | High | Never trust configuration alone; verify every "restricted" binding with an independent external scan. Add host firewall rules as a second layer. |
| Monitoring | No alerting by default. Conpot logs connections internally but does not proactively notify. An unauthenticated write attempt would go unnoticed without manually watching Wireshark. | High | Deploy an IDS or IPS, such as Suricata or Zeek, on a network tap or SPAN port, using a signature equivalent to the tested filter to generate real-time alerts on Modbus write attempts. |

## 3.6 Cleanup

```bash
docker stop conpot && docker rm conpot
```

Completed after all evidence was collected.
