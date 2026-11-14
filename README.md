# Setup Reference

Back to [README](../README.md).

## Working folder

```bash
mkdir -p ~/lab/{iot,mobile,ics,report}
```

## Tools installed and verified on Kali Linux

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y git python3-pip binwalk nmap mosquitto mosquitto-clients mbpoll adb
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

Verified with:

```bash
docker --version
binwalk --help | head -n 3
nmap --version | head -n 1
```

## Additional components

| Component | Location |
|---|---|
| VirtualBox | Host OS, VM isolation layer |
| Android Studio (emulator) | Host OS, not inside the VM |
| Wireshark | Preinstalled on Kali |
| Shodan | Web, free account, passive use only |
