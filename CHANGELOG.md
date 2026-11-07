# Changelog — @babtab/relay

All notable changes to the public npm package `@babtab/relay`
are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [0.3.3] — 2026-11-07

### Added

- Pi now uses native MCP setup via `~/.pi/agent/mcp.json` and the managed stdio bridge; reload Pi with `/reload` or restart it to connect.

## [0.3.2] — 2026-11-07

### Fixed

- Install no longer stalls when the npm registry is slow or unreachable: generated
  MCP configs (setup CLI and every Side Panel one-click link/snippet) now launch
  `npx -y --prefer-offline @babtab/relay@<version> stdio …`.
- Codex: `setup` writes `startup_timeout_sec = 60` so the first-run download is not
  killed by Codex's short default startup timeout.
- Claude Code: re-running `setup` replaces an existing `babtab` entry instead of failing.
- Pairing token file is written atomically (tmp + rename) so a crash cannot wipe tokens.
- HTTP JSON endpoints require `Content-Type: application/json` (415 otherwise), so web
  pages can no longer send preflight-free cross-origin POSTs to the local relay.
- Windows: port-conflict and PATH-check hints use `netstat`/`where.exe` instead of `lsof`/`command -v`.
- Side Panel skips re-rendering when polled approvals/events are unchanged.
- Relay rejects requests whose `Host` header is not loopback (HTTP 403 / WS close 1008) when bound
  to loopback, closing a DNS-rebinding path from web pages.
- Relay answers the extension's app-level `ping` with `pong`; the extension now reconnects when a
  relay that previously answered pings goes silent (half-open sockets after sleep / network change).
- Stdio bridge: if the relay owner process exits mid-session, the surviving bridge restarts the
  relay and retries once on `ECONNREFUSED`, and polls every 500ms while the relay is down.
- Stdio bridge negotiates MCP protocol versions (2024-11-05, 2025-03-26, 2025-06-18) instead of
  always answering 2024-11-05.
- Stdio bridge skips the extra health/status round-trips before a tool call when the background
  poll succeeded within the last 1.5s (lower per-call latency and load).
- Windows: generated MCP configs use `cmd /c npx …` so clients can spawn the `npx.cmd` shim.

- Stdio bridge no longer loops forever on `Waiting for the local bridge to start…`
  when the configured port is held by a frozen or paused process that never
  answers `/health`. After a short grace window (default 8s; override with
  `BABTAB_START_GRACE_MS`) it returns an actionable "port is held / not
  responding" error instead (includes `lsof` probe and a suggested alternate port).

### Changed

- Extension onboarding: louder pairing banner (inline Approve), PATH/Node
  diagnose panel, post-Approve first-command prompt, Try next port (persists
  the new port via the same Save & Connect path) + free-port probe, clearer
  site-access vs Chrome host permission copy, Codex/Antigravity install
  checklists.
- Docs: align deployment claims with the stdio bridge (Node/`npx` required once;
  no longer claim “extension only / no localhost”).

## [0.3.1] — 2026-10-05

### Changed

- Docs: link the published Chrome Web Store listing
  (https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)
  from all 7 READMEs and the privacy page footer; replaced the outdated
  "Load unpacked / pending store publication" steps with store install.
  No code changes — still pairs with extension 0.3.0; no re-pairing needed.
- Install links and setup commands pin `@babtab/relay@0.3.1`.

## [0.3.0] — 2026-10-03

### Added

- Multitab tools: `browser_tabs`, `browser_switch_tab`, `browser_open_tab`,
  `browser_close_tab` (session-scoped; close only touches tabs the session controls).
- `browser_assert`: structured QA assertions over live elements
  (exists / not_exists / text_contains / text_equals / count / visible /
  value_equals / checked). Never throws — failures return ok:false with actuals.
- `browser_extract`: schema-first extraction via fields[].
- Richer target queries: `label`, `placeholder`, and `text` matching plus `nth`
  ordinal selection and `near` proximity hints; documented postcondition
  vocabulary (`element_appeared` / `element_disappeared` are delta checks
  against a pre-dispatch snapshot).
- Token accounting helpers shared by extension and relay (chars/4 estimator,
  stable JSON sizing for prompt-cache friendliness).
- One-click install for Hermes Desktop (`hermes://mcp/install` deeplink);
  one-click copy installers for Codex (`codex mcp add` — the desktop app
  shares the CLI config) and Antigravity (mcp_config.json snippet).
- Protocol unit tests (14) covering the shared extension↔relay contract.

### Changed

- Install links and setup commands pin `@babtab/relay@0.3.0`.
  **Update the relay and extension together** — 0.3.0 pairs with extension 0.3.0.

### Fixed

- Audit-index crash on malformed input.

## [0.2.0] — 2026-09-30

### Automatic local onboarding

- MCP-owned `stdio` bridge starts / reuses a local relay, reconnects after owner
  exit, and stores credentials under `~/.babtab/` instead of the working directory.
- Local `setup` now writes stdio config before any relay or browser connection
  exists. Remote `--relay-url` setup keeps the HTTP pairing flow.
- Tool discovery is immediate; browser calls remain gated by explicit pairing,
  token scopes, and expiry. Interrupted actions are never replayed automatically.
- Extension adds an Add to Cursor link, agent-first onboarding, and an alarm to
  restore opted-in connections after service-worker suspension.
- Publish `@babtab/relay@0.2.0` before releasing extension 0.2.0; its install
  links and setup commands deliberately pin that version.


## [0.1.1] — 2026-09-29

### Security and compatibility

- Device WebSockets now authenticate before registering or approving pairings.
  The extension generates a 256-bit device credential and a `dev_v2_` ID derived
  from its SHA-256 hash; the secret is sent in the first socket frame, not the URL.
  **Update the relay and extension together and re-pair existing agents.**
- The relay binds `127.0.0.1` by default. Remote deployments must explicitly set
  `HOST` and provide TLS. WebSocket clients enforce tool scopes on every request;
  revocation closes existing connections.

### Fixed

- Invalid JSON bodies/envelopes no longer crash the relay; shutdown closes active
  devices and rejects pending requests within a bounded grace period.
- MCP screenshots return image content; oversized text responses retain valid JSON.
- TOML setup replaces nested headers safely, validates output, and atomically
  writes the backed-up config. Values outside babtab are preserved; comments and
  formatting may be normalized by the TOML serializer.

### Added

- `setup` subcommand: `npx -y @babtab/relay setup --target <id> --port <n> --device <id>`
  pairs with the relay (human approves the 6-digit code in the Side Panel)
  and writes the harness MCP config automatically — no hand-editing JSON.
  13 targets: cursor, claude-code (via `claude mcp add`), windsurf, copilot,
  copilot-insiders, codex (TOML), pi (curl probe), claude-desktop + antigravity
  (mcp-remote stdio bridge), devin, kimi, hermes (YAML), manual.
  Existing configs are merged (siblings preserved), JSONC comments tolerated,
  previous file kept as `.bak`, remote relays via `--relay-url`.
- `npx -y @babtab/relay help` and `setup --list-targets`.

## [0.1.0] — 2026-09-28

Initial public release. Run the local bridge without cloning the monorepo:

```bash
npx @babtab/relay
```

### Added

- Local relay: auth (bearer scopes), 6-digit pairing flow, WebSocket
  device routing, Streamable HTTP MCP endpoint (`POST /mcp`).
- Single-file bundle `dist/relay.cjs` (includes `ws`, zero runtime
  dependencies, Node >= 20).
- `babtab-relay` bin + `GET /health` for Side Panel checks.
- Localized READMEs: en, zh-TW, ja, ko, es, pt-BR, hi.
- SECURITY.md and CHANGELOG.md.

### Notes

- Binds `http://127.0.0.1:3000` by default; override with `PORT`.
- Token file via `BABTAB_TOKEN_FILE` (default `./data/relay-tokens.json`).
- Native binaries (`babtab-relay-<os>-<arch>`) ship via GitHub Releases,
  not the npm tarball.
