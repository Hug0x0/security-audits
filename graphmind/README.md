# graphmind — Independent Audit

Code audit of [aouicher/graphmind](https://github.com/aouicher/graphmind): a local-first Rust CLI that builds a knowledge graph of a codebase (tree-sitter + SQLite), with an MCP server for AI assistants, Claude Code / git hooks and a Tauri desktop app.

| | |
|---|---|
| Audited version | v0.3.7, commit [`ccbf1b8`](https://github.com/aouicher/graphmind/commit/ccbf1b8) (2026-10-23) |
| Size | ~30,000 lines excluding tests, 9 Rust crates + React frontend |
| Method | manual static review of the source. Nothing was compiled or run, and no production system was tested. |
| Disclosure | 4 findings withheld pending private disclosure, see [DISCLOSURE.md](DISCLOSURE.md) |

## Verdict

A serious project with solid foundations:
- atomic writes;
- many end-to-end tests;
- file reads through MCP are properly confined;
- no SQL injection;
- signed desktop updater.

The main risks come from two things: the tool changes a lot of configuration on the user's machine, and it re-injects content from untrusted repositories into the AI's context. These risks are covered by the withheld findings. Several functional bugs also make the produced graph wrong.

| Severity | Published | Withheld |
|---|---:|---:|
| Critical / High | 4 | 4 |
| Medium | 16 | — |
| Low | 12 | — |

---

## High

**H-1. `uninstall all` is silently undone by auto-setup.**
- `uninstall_all` resets `setup_version = 0` (`commands/setup.rs:533`).
- On the next command (CLI, git hook or Claude hook), `auto_setup_if_outdated` relaunches `graphmind setup` detached, with output sent to `/dev/null` (`commands/notices.rs:85-127`).
- Shell profiles, `settings.json`, `CLAUDE.md` and MCP configs are then rewritten without consent.

**H-2. Relative imports are never resolved.**
- `Path::new("src").join("./utils")` yields `src/./utils`, which `join` does not normalize. The lookup in `files.path` (`src/utils.ts`) always fails (`graphmind-db/src/builder.rs:703-709`).
- Every import edge falls back to "the first symbol with that name in the project", which creates false edges.

**H-3. UTF-8 panic when truncating MCP responses.**
- `&pretty[..MAX_RESPONSE_BYTES]` (`graphmind-mcp/src/formatting.rs:31,61`) panics when the cut lands inside a multi-byte character.
- Reachable through `gm_fn_impact`, which serializes the full body of every caller without a limit (`handlers/graph.rs:230`).
- Fix: `floor_char_boundary`, and do not serialize `content`.

**H-4. An existing `pre-push` hook can stop blocking.**
- `install_hook` appends graphmind's code to the end of an existing hook (`commands/hooks.rs:142-148`), and that code ends with `echo ""`. The hook therefore always exits 0: a failing `npm test` no longer blocks the push.
- Conversely, with `set -e`, a missing `graphmind` blocks every push.

## Medium

**M-1. `.tsx` files are parsed without JSX.** `graphmind-core/src/parser.rs:15` uses `LANGUAGE_TYPESCRIPT` instead of `LANGUAGE_TSX`. React components produce ERROR nodes.

**M-2. JS/TS arrow functions are never counted as callers.** Only `function_declaration` and `method_definition` are (`extractor.rs:326-331`).

**M-3. Python methods are extracted twice**, as `Method` then as `Function` (`languages/python.rs:52-63`).

**M-4. Graph build.**
- Pass 1 runs without a transaction (`builder.rs:284-439`): it is slow, and an MCP reader sees a half-rebuilt graph.
- In incremental builds, callers in unchanged files are not re-resolved.

**M-5. Unescaped `LIKE` wildcards.**
- `gm_fn symbol="%"` matches every symbol and returns their content (`queries.rs:132-139`).
- Definitions are not subject to `limit`.

**M-6. A negative limit removes any bound.**
- In SQLite, `LIMIT -N` means "unlimited". Affects `gm_search`, `gm_dead_code`, `gm_map` and `gm_similar`.
- Fix: `clamp(1, MAX)`.

**M-7. MCP responses without a size cap.** `gm_file`, `gm_outline`, `gm_deps`, `gm_export` (unbounded depth), `gm_impact` and `gm_diff_impact` bypass `MAX_RESPONSE_BYTES`.

**M-8. Inconsistent isolation between projects.**
- Without `project`, `gm_query`, `gm_fn` and `gm_search` query **every** registered repository and return its source code (`handlers/graph.rs:54-112`).
- `gm_file`, on the other hand, is confined to the project.

**M-9. `hooks uninstall` is destructive.**
- It deletes every line containing "graphmind", including the user's own. The write is not atomic and has no backup.
- It leaves a syntactically invalid `post-checkout` (empty `if … then`) (`hooks.rs:321-334`).

**M-10. Unescaped paths.**
- Claude hook commands (`claude_hook.rs:520,548`): a HOME containing a space breaks the hooks.
- Shell profile line (`setup/shell_path.rs:51,109`).
- Single-quoted PowerShell strings: a username like `O'Brien` breaks the command (`shell_path.rs:206,243,354`).

**M-11. `*.bak` files pile up without limit** (`graphmind-config/src/atomic.rs:39`). Some, such as `.vscode/mcp.json.<ts>.bak`, are created inside the repository's working tree and may get committed.

**M-12. `update` deletes `~/.local/bin/graphmind`**, even though it is `install.sh`'s default install path (`update.rs:60-68`). It also compares versions with `==`, so a user on an `-rc` gets downgraded.

**M-13. Embeddings.**
- `cosine_similarity` computes over `min(len)` (`search.rs:19`): vectors of different dimensions, after a provider switch, produce meaningless scores without any error.
- `local.rs:46` loads nomic (768 dimensions) but labels it `all-MiniLM-L6-v2` (384).
- Embeddings of deleted memories are never removed, text included: `delete_by_symbol` is never called.

**M-14. "No cloud, no telemetry" only holds by default.**
- In OpenAI or Voyage mode, paths, signatures and the first 512 characters of each function body are sent.
- A license sends a JWT and a fingerprint (`SHA256(hostname|user|os)`) to `graphmind-server.fly.dev` every 24 hours.
- The MCP server queries the GitHub API on startup, with no opt-out.
- All of this should be documented.

**M-15. Desktop app.**
- No CSP, and `withGlobalTauri: true` (`tauri.conf.json`). No XSS was found, but the IPC surface is powerful.
- Synchronous commands run on the main thread (the UI freezes).
- Two concurrent builds can run on the same database.

**M-16. Rate limiting.** A 429 is treated as an oversized batch (`resilient.rs:83-107`): there is no backoff and no `Retry-After` handling, and the number of requests grows.

## Low

- Recursive AST walks with no depth limit and no tree-sitter timeout: a stack overflow is possible on a hostile file.
- JSONL store: append errors are ignored (`store.rs:326,571`), and an `expect` can bring down the MCP server. `cross_links.rs` can lose the file when the disk is full.
- `callers` and `callees` filter confidence inconsistently. `gm_export` duplicates edges, and its Mermaid/DOT output is not escaped.
- `gm_diff_impact` compares repository-relative paths with project-relative paths.
- `CLAUDE.md` uninstall: if the `END` marker is missing, everything after `START` is deleted.
- `install_skill` replaces skill folders with generic names without checking their origin.
- `exclude --global` changes the config without taking the lock.
- Memory consolidation is O(n²), and tags are lost on deduplication.
- License fingerprint: `&fingerprint[..8]` can panic, and `hostname` is resolved through PATH.
- CI: actions referenced by tag rather than SHA, no `cargo audit` or `cargo deny`, desktop app excluded from clippy and tests, no macOS or Windows tests, unpinned toolchain.
- `scripts/build-desktop.sh` hardcodes a version and does not sign the DMG. The versions in `tauri.conf.json` and `package.json` are out of sync.
- Code quality: 750–960 line files, about 200 lines of bash embedded in Rust strings without tests, many `.ok()` calls that swallow errors.

## Strengths

- `gm_file` confinement (`graphmind-db/src/paths.rs:32-53`):
  - only `Normal` path components are accepted;
  - the path is canonicalized and the target must be indexed;
  - FIFOs and directories are refused;
  - dedicated tests cover symlinks and FIFOs.
- Fully parameterized SQL, quoted FTS5 terms, WAL, migrations under `BEGIN IMMEDIATE`.
- `write_atomic` (fsync + rename). An unreadable or JSONC file is never rewritten.
- Husky and `core.hooksPath` are detected, and inter-process locks are in place.
- Minisign-signed desktop updater, Apple notarization in CI, up-to-date dependencies (tauri 2.11, rusqlite 0.32).

## Disclaimer

Independent, best-effort static review, not affiliated with graphmind or its author. A finding is not proof of exploitability, and the absence of a finding does not mean the absence of a bug. Any vulnerability should be reported privately to `security@getgraphmind.com` (see the project's [SECURITY.md](https://github.com/aouicher/graphmind/blob/main/SECURITY.md)), not in a public issue.
