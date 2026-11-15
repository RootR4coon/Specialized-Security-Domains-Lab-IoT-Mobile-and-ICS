# Screenshot Inventory

Rename each original file to the corresponding target filename and place it in the folder shown, matching the image references used throughout the documentation.

## screenshots/00-setup/

| Original filename | Target filename |
|---|---|
| `01_upgradeKali.png` | `00-setup-kali-upgrade.png` |
| `02_docker_ps_isworking.png` | `00-setup-docker-ps.png` |
| `03_docker_binwalk_nmap.png` | `00-setup-tools-verified.png` |
| `04_Kali_active.png` | `00-setup-vm-active.png` |
| `05_iot.png` | `00-setup-folders.png` |

## screenshots/01-iot/

| Original filename | Target filename |
|---|---|
| `06_SHODAN_mosquitto.png` | `01-iot-shodan-mosquitto-results.png` |
| `07_binwalk_firmware_analysis.png` | `03-iot-binwalk-identify.png` |
| `11_squashfs-root.png` | `04-iot-binwalk-extract.png` |
| `08_theRootFileSystem.png` | `04b-iot-squashfs-contents.png` (verify content before placing) |
| `12_MD5-crypt_ie_weak_hashing_algorithm.png` | `05-iot-passwd-shadow.png` |
| `13_The$1$_MD5_lines.png` | `05-iot-passwd-shadow.png` (alternate/detail view, or combine with the above) |
| `14-JohnTheRipper.png` | `14-JohnTheRipper.png` |
| `15-PasswordCRACKED.png` | `15-PasswordCRACKED.png` |
| `09_SearchForAnyPasswd_orSecret.png` | `09-iot-search-secrets.png` |
| `09_Search_Weaknesses.png` | `09b-iot-search-secrets-continued.png` |
| `10_StartupScripts_anythingSuspicious.png` | `10-iot-init-scripts.png` |
| `10-mosquitto_unlocked.png` | `10-iot-mosquitto-subscriber.png` |
| `11-Wireshark_mosquitto.png` | `11-iot-wireshark-mqtt-plaintext.png` |
| `12-PublishMessage_LivingRoomTemp.png` | (supporting evidence; not individually referenced) |
| `13-Wireshark-TCP_port8883.png` | `13-iot-wireshark-tls-encrypted.png` |
| `14-TLS-error.png` | `12-iot-anonymous-denied.png` |
| `16-incrementalInstallSuccess.png` | Not yet mapped; confirm content |

## screenshots/02-mobile/

| Original filename | Target filename |
|---|---|
| `21-emulator.png` | `01-mobile-emulator-running.png` |
| `17-MobSFv4.6.png` | `17-mobile-mobsf-home.png` |
| `18-localhost_8000.png` | `18-mobile-mobsf-browser.png` |
| `20_MobSF_Scoreboard.png` | `20-mobile-mobsf-scoreboard.png` |
| `frida_communicates_with_myEmulator.png` | `24-mobile-frida-ps.png` |
| `19-uncrackable_success.png` | `19-mobile-uncrackable.png` |
| `22_InstallDrozerClient.png` | `22-mobile-drozer-client.png` |

## screenshots/03-ics/

| Original filename | Target filename |
|---|---|
| `22-ConpotContainer.png` | `22-ics-conpot-container.png` |
| `25-mmodus_tcp1.png` | `25-ics-wireshark-modbus-read.png` |
| `26-modus_tcp2.png` | `26-ics-wireshark-modbus-write.png` |
| `27_modbusCommand_wireshark.png` | `27-ics-wireshark-write-filter.png` |
| `27-silentlyBroadensAll*interfaces.png` | `27-ics-lsof-interfaces.png` |
