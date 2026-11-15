# 2. Mobile Security

Back to [README](../README.md). Findings table: [notes/mobile-findings.md](../notes/mobile-findings.md).

## 2.1 Device Preparation

Android Studio AVD created (Pixel device, Google APIs system image) and launched. `adb devices` confirmed the emulator as authorized and connected.

[Figure: The emulator.]
<img src="https://github.com/user-attachments/assets/0b1033b1-8f8a-4c20-b8fe-583476a48ca1" alt="The emulator" width="150">

**Note on a resolved setup issue:** Earlier in this lab, ADB repeatedly reported `device unauthorized`, with the on-device confirmation dialog disappearing before it could be approved. This was resolved by regenerating the ADB key pair, revoking old USB debugging authorizations on the device, and cold-booting the AVD (rather than resuming from a snapshot) before reconnecting.

## 2.2 Static Analysis with MobSF

Ran MobSF via Docker:

```bash
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Uploaded the DIVA APK and reviewed the generated report.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/40f29bab-4e0c-4b54-b749-8e6cdc5d9e0e"
      <br><sub>MobSFv4.6</sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ae2bb1ad-073a-42d6-ac3e-cc7a9883e85b"
      <br><sub>localhost 8000</sub>
   <td align="center">
      <img src="https://github.com/user-attachments/assets/ceae2fdc-5ac4-4638-8f18-b765c44d353e" 
      <br><sub>MobSF Scoreboard</sub>
    <td align="center">
    </td>
  </tr>
</table>

| Report section | Finding |
|---|---|
| Permissions | [fill in — dangerous permissions the app does not need] |
| Manifest Analysis | [fill in — debuggable, allowBackup, exported components] |
| Code Analysis | [fill in — hardcoded secrets, weak crypto, insecure storage] |
| Network Security | [fill in — cleartext traffic allowed, certificate validation] |
| MobSF security score | [fill in] |

## 2.3 Dynamic Testing with Frida and Objection

Confirmed Frida server and client communication:

```bash
frida-ps -U
```

Returned the live process list from the emulator.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/261115aa-eec8-4f58-a4ef-580a01417013"
      <br><sub>Frida communicates with my Emulator</sub>
   <td align="center">
    </td>
  </tr>
</table>

Explored the UnCrackable Level 1 app with Objection:

```bash
objection -g owasp.mstg.uncrackable1 explore
```

Ran `android hooking list activities`, `android hooking list classes`, `android keystore list`, and `env` inside the session.

**Learning point confirmed:** UnCrackable Level 1 performs root detection. The intended exercise, locating a hidden secret, is documented in the official OWASP MASTG write-up.

## 2.4 Component Assessment with Drozer

Installed the Drozer client (`pipx install drozer`, version 3.1.0, Python 3.9.6) and the Drozer Agent APK (current releases maintained under `ReversecLabs/drozer-agent`, following the project's rename from WithSecureLabs/mwrlabs).

```bash
adb forward tcp:31415 tcp:31415
drozer console connect
run app.package.list -f diva
run app.package.attacksurface jakhar.aseem.diva
run app.activity.info -a jakhar.aseem.diva
```

| Component type | Exported count |
|---|---|
| Activities | [fill in] |
| Services | [fill in] |
| Providers | [fill in] |

**Why this matters:** Exported components without permission checks can be invoked directly by any other application on the device, or via `adb`, bypassing the app's intended entry points and potentially exposing sensitive functionality or data.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/261115aa-eec8-4f58-a4ef-580a01417013"
      <br><sub>Uncrackable success</sub>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/c1a96fc9-adc7-438d-a83b-afd3544506fc"
      <br><sub>Install Drozer Client</sub>
    </td>
  </tr>
</table>

## 2.5 Authentication Review and MASVS Mapping

| Checklist item | Finding | Evidence |
|---|---|---|
| Credentials stored in plain text (shared preferences, SQLite, files) | [fill in] | |
| Credentials written to logs (`adb logcat \| grep -i pass`) | [fill in] | |
| Hardcoded keys or tokens in code | [fill in; cross-reference MobSF Code Analysis] | |
| Login works without server-side validation | [fill in] | |
| Sensitive data copied to clipboard or backups | [fill in] | |

### Secure Development Practices (OWASP MASVS)

| MASVS Area | Secure Practice |
|---|---|
| Storage | Use Android Keystore and encrypted storage; never plain text |
| Crypto | Use modern algorithms; no hardcoded keys |
| Auth | Validate on the server; use short-lived tokens; use biometric prompts |
| Network | Enforce TLS with certificate validation; disable cleartext traffic |
| Platform | Do not export components unnecessarily; set `allowBackup=false` |
| Resilience | Disable debugging in release builds |
