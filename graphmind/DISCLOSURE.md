# Disclosure status

Four findings have real impact on users. Their technical details are **withheld** from this public repository until both conditions below are met:
- they have been reported privately to the maintainer at `security@getgraphmind.com`, as the project's [SECURITY.md](https://github.com/aouicher/graphmind/blob/main/SECURITY.md) requests;
- a fix has shipped, or 90 days have passed.

| ID | Area | Severity | Status |
|---|---|---|---|
| FS-01 | Directory deletion outside the graphmind folder (CLI and desktop) | Critical | Reported privately 2026-10-24 |
| SUPPLY-01 | Binary integrity on install and update | High | Reported privately 2026-10-24 |
| INJ-01 | Untrusted content re-injected into the AI assistant's context | High | Reported privately 2026-10-24 |
| PERM-01 | Permissions of the configuration file holding secrets | Medium | Reported privately 2026-10-24 |

Full details will be published once a fix ships, or on 2027-01-22 at the latest.

All other findings are robustness, correctness or quality issues, and are published in full in the [report](README.md).
