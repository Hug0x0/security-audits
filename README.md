# Security audits

Independent code audits of open-source projects by [Hug0x0](https://github.com/Hug0x0).

| Project | Type | Date | Verdict | Report |
|---|---|---|---|---|
| [aouicher/graphmind](https://github.com/aouicher/graphmind) | Rust CLI + MCP server + Tauri app | 2026-10-23 | 4 high published, 4 withheld pending disclosure (1 critical) | [graphmind/](graphmind/README.md) |
| [marclou/mailcheap](https://github.com/marclou/mailcheap) | Next.js / MongoDB SaaS | 2026-10-21 | 0 critical, 1 high (ops), 21 medium, 2 withheld | [mailcheap/](mailcheap/README.md) |
| [Aboudjem/Sleepless](https://github.com/Aboudjem/Sleepless) | macOS Swift app (sudoers) | 2026-10-16 | Safe, 2 minor notes | [sleepless/](sleepless/README.md) |

## Responsible disclosure

Findings with real impact on users are first reported privately to the maintainer, through their security policy or GitHub private vulnerability reporting. Their details are published only once a fix has shipped, or 90 days after the report. Each audit's `DISCLOSURE.md` gives the current status.

## Method

Manual static review of the source code, assisted by tooling (`npm audit`, targeted searches, review agents). No production system is tested. Each report states the audited commit.

## Disclaimer

These audits are independent, best-effort reviews, not affiliated with any of the audited projects. A finding is not proof of exploitability, and the absence of a finding does not mean the absence of a bug.
