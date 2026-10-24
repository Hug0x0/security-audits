# Sleepless — Independent Audit

Security audit of [Aboudjem/Sleepless](https://github.com/Aboudjem/Sleepless), a macOS menu-bar app that keeps the Mac awake with the lid closed using `pmset disablesleep`.

| | |
|---|---|
| Audited version | v1.2.7, `main` branch as of 2026-10-16 |
| Scope | `App.swift` (705 lines), the 4 shell scripts, the sudoers rule, the 2 CI workflows, `docs/index.html` and the Homebrew cask |
| Method | manual static review of the source. The released binary was not compared against the source. |
| Follow-ups | [Aboudjem/Sleepless#10](https://github.com/Aboudjem/Sleepless/issues/10) and [#11](https://github.com/Aboudjem/Sleepless/pull/11) (macOS 13+ support), [Aboudjem/homebrew-tap#1](https://github.com/Aboudjem/homebrew-tap/pull/1) |

## Verdict: safe, nothing malicious

**What the app does**
- It runs only three system binaries:
  - `pmset`, to read sleep and battery state;
  - `sudo pmset -a disablesleep 0/1`;
  - `osascript`, for notifications and the one-time rule installation.
- It never touches the network: no HTTP calls, no telemetry, no auto-update.
- It installs no root daemon or LaunchDaemon. The only addition is an optional login item, off by default.
- Several safeguards turn the awake mode off: battery floor, Low Power Mode, end of the timer, and a reset on every reboot.
- The audit found no obfuscated code, no base64, no hidden Unicode characters and no prebuilt binary (apart from the `.icns` icon).

**Sudo rule**
- It is minimal: it only allows `pmset -a disablesleep 0` and `pmset -a disablesleep 1`, with exactly fixed arguments.
- It is validated with `visudo -c` before installation.
- `uninstall.sh` removes it and checks that it is gone.

**Build chain**
- The binary is built on GitHub Actions, with a SLSA (Sigstore) attestation and SHA-256 checksums.
- The Homebrew cask downloads the official release.

## Notes (minor)

| ID | Severity | Finding |
|---|---|---|
| L-1 | Low | Any program running in the user session can keep the Mac awake without a password. That is the purpose of the rule. Worst case, the Mac heats up or drains its battery, but no privilege escalation is possible. |
| L-2 | Low | When the user clicks "Enable", `grant.sh` runs as root from inside the app bundle. Malware already present in the session could modify that script before authentication. This assumes an already compromised machine, and macOS partly protects apps in `/Applications`. |
| I-1 | Info | The app is not notarized (ad hoc signature), so users must go through "Open Anyway". |
| I-2 | Info | The released binary was not compared against the source. To check it, run `gh attestation verify` on the zip, or build it yourself with `./install.sh`. |

At the time of the audit, the cask required macOS 26. The PRs listed above propose lowering this to macOS 13+.

## Disclaimer

Independent, best-effort static review, not affiliated with the project. The absence of a finding does not mean the absence of a bug.
