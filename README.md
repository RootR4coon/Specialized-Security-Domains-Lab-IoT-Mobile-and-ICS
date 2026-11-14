# Mobile Findings

Detailed methodology and evidence: [docs/02-mobile.md](../docs/02-mobile.md).

| ID | Title | Severity | Key evidence | Recommendation |
|---|---|---|---|---|
| MOB-01 | MobSF static analysis of DIVA completed | TBD | `17-mobile-mobsf-home.png`, `20-mobile-mobsf-scoreboard.png` | Complete the per-section table in docs/02-mobile.md |
| MOB-02 | Frida server and client communication confirmed with emulator | Informational | `24-mobile-frida-ps.png` | N/A (setup verification) |
| MOB-03 | Objection session established against UnCrackable Level 1; root detection confirmed | Informational | `19-mobile-uncrackable.png` | Follow official MASTG write-up for the hidden-secret exercise |
| MOB-04 | Drozer client installed and console connected to Agent | Informational | `22-mobile-drozer-client.png`, `23-mobile-drozer-connected.png` | Complete exported-component counts |

## Authentication review checklist

| Checklist item | Finding |
|---|---|
| Credentials stored in plain text | [fill in] |
| Credentials written to logs | [fill in] |
| Hardcoded keys or tokens in code | [fill in] |
| Login works without server-side validation | [fill in] |
| Sensitive data copied to clipboard or backups | [fill in] |

## MASVS mapping

| MASVS Area | Secure Practice |
|---|---|
| Storage | Android Keystore and encrypted storage; never plain text |
| Crypto | Modern algorithms; no hardcoded keys |
| Auth | Server-side validation; short-lived tokens; biometric prompts |
| Network | TLS with certificate validation; disable cleartext traffic |
| Platform | No unnecessary exported components; `allowBackup=false` |
| Resilience | Disable debugging in release builds |
