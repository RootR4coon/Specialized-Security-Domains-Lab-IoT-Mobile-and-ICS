# 1. IoT Security

Back to [README](../README.md). Findings table: [notes/iot-findings.md](../notes/iot-findings.md).

## 1.1 Shodan Exposure Assessment (Passive)

Search query: `product:mosquitto`

| Item | Result |
|---|---|
| Total result count | [fill in from your screenshot] |
| Top 3 countries | [fill in from Top Countries panel] |
| Own public IP search result | [fill in — exposed or nothing found] |

**Risk note:** Default ports with no authentication allow any internet-wide scanner, not just a targeted attacker, to discover and potentially interact with exposed IoT devices at scale. This is the same exposure pattern that allowed the Mirai botnet to compromise large numbers of devices automatically.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/6ccbd7ff-08d5-48d1-8dc1-3d81fb499978" alt="Shodan search results" width="450"><br>
      <sub>Shodan search results for product:mosquitto, showing total count and Top Countries panel.</sub>
    </td>
  </tr>
</table>

## 1.2 Firmware Analysis (OWASP IoTGoat)

**Target:** `IoTGoat-raspberry-pi2.img`, obtained from the OWASP IoTGoat GitHub Releases page.

**Identification (`binwalk`):** Identified a flattened device tree and a SquashFS filesystem at offset `29360128` (`0x1C00000`), little endian, version 4.0, xz compression, 3,946,402 bytes, 1,333 inodes, blocksize 262,144 bytes, created 2019-01-30.

**Extraction:** `binwalk -e IoTGoat-raspberry-pi2.img` extracted the filesystem to `_IoTGoat-raspberry-pi2.img.extracted/squashfs-root`.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ef6efcf7-669f-489a-a921-a7879ebc7eda" width="380">
      <br><sub>Binwalk output identifying the Linux kernel and SquashFS filesystem inside the firmware image.</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/686b2ac8-5b25-4954-9aaa-560c14142a0b" 
      <br><sub>Extraction complete, with the squashfs-root path located.</sub>
    </td>
  </tr>
</table>

### Finding IOT-01: Weak Password Hashing Algorithm (MD5-crypt)

- **File:** `etc/shadow`
- **Detail:** The `root` account hash begins with `$1$` (MD5-crypt) rather than a modern algorithm such as `$6$` (SHA-512-crypt). A second account, `iotgoatuser`, also uses `$1$`.
- **Impact:** MD5-crypt is computationally inexpensive to attack offline. Because this hash is embedded in the firmware image, it is identical across every device running the same firmware version.
- **Severity:** High
- **Recommendation:** Use a modern, salted, computationally expensive hash (SHA-512-crypt, bcrypt, or Argon2) for all stored credentials. Rotate credentials per device rather than relying on a firmware-wide default.
- **Reference:** OWASP IoT Top 10 — Weak, Guessable, or Hardcoded Passwords

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/6bebed22-162a-48bc-8023-f6368cc43885"
      <br><sub>MD5 crypt i.e. Weak Hashing Algorithm</sub>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/11b181c2-22d4-4985-ae95-25128c4ab474"
      <br><sub>The "$1$" MD5 lines</sub>
    </td>
  </tr>
</table>

### Finding IOT-02: Default Credential Recovered

- **Detail:** Using the Mirai botnet credentials wordlist, bundled with Metasploit at `/usr/share/metasploit-framework/data/wordlists/mirai_pass.txt`, against the extracted `etc/shadow` hash with John the Ripper:

  ```bash
  john --wordlist=/usr/share/metasploit-framework/data/wordlists/mirai_pass.txt /tmp/iotgoat_hashes.txt
  ```

  Result: the `iotgoatuser` password was recovered as `7uJMko0vizxv` in under one second against a 62-entry wordlist. This is a documented default credential originating from the leaked Mirai botnet source code, reused intentionally in this training firmware.
- **Note:** The `root` account hash was not recovered using this wordlist, nor a general 14-million-entry wordlist (`rockyou.txt`). Public write-ups of this exercise indicate root access is intended to be obtained through a different vulnerability, not password cracking.
- **Impact:** Default or reused IoT credentials are the exact mechanism that allowed the real Mirai botnet to compromise large numbers of devices automatically and at scale.
- **Severity:** Critical
- **Recommendation:** Never ship devices with default or hardcoded credentials. Force a credential change on first boot. Disable or randomize any built-in test or maintenance accounts before production release.
- **Reference:** OWASP IoT Top 10; NIST IoT Security Program

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/f0091ac0-b50e-46a7-8af8-24805fd2ff7b" />
      <br><sub>Search for any password or secret</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/c8b57aa6-5664-4378-a034-e16fbb36cdfd" />
      <br><sub>John The Ripper</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/b1a4aa66-7e8b-47b8-9a1f-88083579fc46" />
      <br><sub>Password CRACKED</sub>
    </td>
  </tr>
</table>

### Finding IOT-03: Hardcoded Secrets, Configuration Files, and Startup Services

Checklist performed per the lab's "what to look for" guidance:

```bash
grep -ri 'password' etc | head -n 20
find . -name '*.conf' | head -n 20
find . -name '*.pem' -o -name '*.key'
ls etc/init.d
```

| Result | Detail |
|---|---|
| Hardcoded password references in config files | [fill in from your screenshot] |
| Private keys (.pem, .key) found | [fill in] |
| Unnecessary services in etc/init.d | [fill in — e.g. telnet, debug shells] |

- **Impact:** Anyone with access to the firmware image can read these files directly, without needing to compromise a running device.
- **Severity:** [fill in based on actual findings — typically Medium to High]
- **Recommendation:** Remove unnecessary services before production release; never store secrets in plaintext configuration files within the firmware image; use a hardware-backed secure element for key storage where possible.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/5720a7a4-d614-4cfd-a780-dc2eb0d31dbd" alt="Mosquitto UNLOCKED" width="450"><br>
      <sub>Mosquitto UNLOCKED</sub>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/f076fa68-fbd7-491a-a638-cb467ca54a71" alt="Wireshark mosquito" width="450"><br>
      <sub>Wireshark mosquito</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/3846923d-b478-4217-bf7d-379f5bb08a0c" alt="PublishMessage living room's temperature" width="450"><br>
      <sub>PublishMessage living room's temperature</sub>
    </td>
  </tr>
</table>

## 1.3 MQTT Protocol Test (Plaintext)

Started a subscriber and published test messages:

```bash
mosquitto_sub -h localhost -t 'home/#' -v
mosquitto_pub -h localhost -t home/livingroom/temp -m 21.5
mosquitto_pub -h localhost -t home/door/lock -m unlocked
```

Both messages arrived at the subscriber in plaintext. Captured in Wireshark on the loopback interface with filter `mqtt`; the topic and message payload were fully readable inside the PUBLISH packet.

**Fields visible in plaintext:** topic string, message payload, QoS level, packet type.

**What an attacker on the same network could learn:** exact sensor readings (for example, temperature) and device state changes (for example, door lock status) in real time, without needing to break any encryption.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/199f9290-9a2d-4d7f-9341-d4f37c3fcd94"
      <br><sub>Mosquitto unlocked</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/88087ea4-0ab8-4beb-8cb6-43e274ae3f76"
      <br><sub>Wireshark mosquitto</sub>
    </td>
  </tr>
</table>

### Finding IOT-04: MQTT Broker Accepts Unauthenticated, Unencrypted Connections by Default

- **Impact:** Any device on the network segment can publish or subscribe to any topic, and any passive observer can read all traffic, including sensitive state such as door lock status.
- **Severity:** High
- **Recommendation:** Addressed directly in Section 1.4 below.
- **Reference:** OWASP IoT Top 10 — Insecure Network Services

## 1.4 MQTT Security Controls

Implemented controls:

1. Created a dedicated MQTT user: `mosquitto_passwd -c /etc/mosquitto/passwd iotuser`
2. Generated a self-signed TLS certificate:

   ```bash
   openssl req -x509 -newkey rsa:2048 -nodes -keyout server.key -out server.crt \
     -days 365 -subj '/CN=localhost' -addext 'subjectAltName=DNS:localhost'
   ```

3. Added a secure listener configuration (see `configs/secure.conf` in this repository):

   ```conf
   listener 8883
   allow_anonymous false
   password_file /etc/mosquitto/passwd
   certfile /etc/mosquitto/certs/server.crt
   keyfile /etc/mosquitto/certs/server.key
   ```

4. Restarted the broker and confirmed `active (running)` via `systemctl status`.

**Verification:**

| Test | Result |
|---|---|
| Unauthenticated, unencrypted attempt on port 8883 | Failed, as expected |
| Authenticated attempt with `--cafile`, `-u iotuser`, `-P <password>` | Succeeded |
| Wireshark capture on `tcp.port == 8883` after the fix | Payload now appears as encrypted TLS Application Data; not readable |

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/d051d131-fefa-44a1-bc26-04a67778595d"
      <br><sub>TLS error</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/fb8aed9f-48b2-422f-a62d-fd6a161f54b2"
      <br><sub>Wireshark TCP</sub>
    </td>
  </tr>
</table>

**Before and after comparison** (strongest evidence in this section):

| Before (Section 1.3) | After (Section 1.4) |
|---|---|
| Topic and message readable in plain text | Payload unreadable, encrypted with TLS |
| `screenshots/01-iot/11-iot-wireshark-mqtt-plaintext.png` | `screenshots/01-iot/13-iot-wireshark-tls-encrypted.png` |
