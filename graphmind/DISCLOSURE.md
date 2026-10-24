# Disclosure status

Four findings have real impact on users. Their technical details are **withheld** from this public repository until both conditions below are met:
- they have been reported privately to the maintainer at `security@getgraphmind.com`, as the project's [SECURITY.md](https://github.com/aouicher/graphmind/blob/main/SECURITY.md) requests;
- a fix has shipped, or 90 days have passed.

| ID | Area | Severity | Status |
|---|---|---|---|
| FS-01 | Directory deletion outside the graphmind folder (CLI and desktop) | Critical | Private report pending |
| SUPPLY-01 | Binary integrity on install and update | High | Private report pending |
| INJ-01 | Untrusted content re-injected into the AI assistant's context | High | Private report pending |
| PERM-01 | Permissions of the configuration file holding secrets | Medium | Private report pending |

All other findings are robustness, correctness or quality issues, and are published in full in the [report](README.md).
