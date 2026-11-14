# 2. Mobile Security

Back to [README](../README.md). Findings table: [notes/mobile-findings.md](../notes/mobile-findings.md).

## 2.1 Device Preparation

Android Studio AVD created (Pixel device, Google APIs system image) and launched. `adb devices` confirmed the emulator as authorized and connected.

![Figure: Android emulator running and authorized via adb devices.](../screenshots/02-mobile/01-mobile-emulator-running.png)
*Figure: Android emulator running and authorized via adb devices.*

**Note on a resolved setup issue:** Earlier in this lab, ADB repeatedly reported `device unauthorized`, with the on-device confirmation dialog disappearing before it could be approved. This was resolved by regenerating the ADB key pair, revoking old USB debugging authorizations on the device, and cold-booting the AVD (rather than resuming from a snapshot) before reconnecting.

## 2.2 Static Analysis with MobSF

Ran MobSF via Docker:

```bash
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Uploaded the DIVA APK and reviewed the generated report.

![Figure: MobSF running and ready for upload.](../screenshots/02-mobile/17-mobile-mobsf-home.png)
*Figure: MobSF running and ready for upload.*

![Figure: MobSF interface open in the browser at localhost:8000.](../screenshots/02-mobile/18-mobile-mobsf-browser.png)
*Figure: MobSF interface open in the browser at localhost:8000.*

![Figure: MobSF security scoreboard for the analyzed APK.](../screenshots/02-mobile/20-mobile-mobsf-scoreboard.png)
*Figure: MobSF security scoreboard for the analyzed APK.*

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

![Figure: frida-ps -U output, confirming Frida server and client communication with the emulator.](../screenshots/02-mobile/24-mobile-frida-ps.png)
*Figure: frida-ps -U output, confirming Frida server and client communication with the emulator.*

Explored the UnCrackable Level 1 app with Objection:

```bash
objection -g owasp.mstg.uncrackable1 explore
```

Ran `android hooking list activities`, `android hooking list classes`, `android keystore list`, and `env` inside the session.

**Learning point confirmed:** UnCrackable Level 1 performs root detection. The intended exercise, locating a hidden secret, is documented in the official OWASP MASTG write-up.

![Figure: Objection session exploring the UnCrackable Level 1 app.](../screenshots/02-mobile/19-mobile-uncrackable.png)
*Figure: Objection session exploring the UnCrackable Level 1 app.*

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

![Figure: Drozer client installation.](../screenshots/02-mobile/22-mobile-drozer-client.png)
*Figure: Drozer client installation.*

![Figure: Drozer console connected to the Agent on the emulator.](../screenshots/02-mobile/23-mobile-drozer-connected.png)
*Figure: Drozer console connected to the Agent on the emulator.*

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
