# Setup Reference
Back to [README](../README.md).
<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/c7b7cc55-17e4-41aa-9535-a02b23d784df"
      <br><sub>Kali system upgraded</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/082f396a-382e-46bc-ba59-6dbaa27c139c"
      <br><sub>Kali VM running</sub>
    </td>
  </tr>
</table>

*Figure: Kali Linux virtual machine running in the isolated lab environment.*

## Working folder

```bash
mkdir -p ~/lab/{iot,mobile,ics,report}
```
[Figure: Lab directory structure created for IoT, Mobile, ICS and reporting activities.]<img width="194" height="50" alt="05_iot" src="https://github.com/user-attachments/assets/47261a53-369c-4992-8c4b-cbb2498e98f3" />

*Figure: Initial folder structure created for the security assessment.*
## Tools installed and verified on Kali Linux

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y git python3-pip binwalk nmap mosquitto mosquitto-clients mbpoll adb
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

*Figure: Docker environment verified and running.*

Verified with:

```bash
docker --version
binwalk --help | head -n 3
nmap --version | head -n 1
```
<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/4d326e9e-b67b-41d8-9fda-e0365790d22d"
      <br><sub>Docker ps is working</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ee47f0a8-b293-4f82-9bc4-e7f61822a5b4"
      <br><sub>Docker Binwalk and Nmap</sub>
    </td>
  </tr>
</table>

## Additional components

| Component | Location |
|---|---|
| VirtualBox | Host OS, VM isolation layer |
| Android Studio (emulator) | Host OS, not inside the VM |
| Wireshark | Preinstalled on Kali |
| Shodan | Web, free account, passive use only |
