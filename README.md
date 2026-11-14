# IoT Findings

Detailed methodology and evidence: [docs/01-iot.md](../docs/01-iot.md).

| ID | Title | Severity | Key evidence | Recommendation |
|---|---|---|---|---|
| IOT-01 | Weak password hashing algorithm (MD5-crypt) in `etc/shadow` for `root` and `iotgoatuser` | High | `05-iot-passwd-shadow.png` | Use modern salted, expensive hashes (SHA-512-crypt, bcrypt, Argon2); per-device credential rotation |
| IOT-02 | Default credential recovered: `iotgoatuser` cracked in under one second with the Mirai wordlist | Critical | `14-JohnTheRipper.png`, `15-PasswordCRACKED.png` | Never ship default credentials; force change on first boot; disable test accounts |
| IOT-03 | Hardcoded secrets, config files, and startup services in firmware image | TBD | `09-iot-search-secrets.png`, `10-iot-init-scripts.png` | Remove unnecessary services; no plaintext secrets in images; hardware-backed key storage |
| IOT-04 | MQTT broker accepts unauthenticated, unencrypted connections by default | High | `10-iot-mosquitto-subscriber.png`, `11-iot-wireshark-mqtt-plaintext.png` | TLS on 8883, `allow_anonymous false`, per-client credentials (verified in Section 1.4) |

## Control verification

| Test | Result |
|---|---|
| Anonymous publish on 8883 | Rejected, as expected |
| Authenticated publish with `--cafile` and `-u` | Succeeded |
| Wireshark on `tcp.port == 8883` | Payload unreadable TLS Application Data |
