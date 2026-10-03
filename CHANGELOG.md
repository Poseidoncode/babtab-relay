# Changelog — @babtab/relay

All notable changes to the public npm package `@babtab/relay`
are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [Semantic Versioning](https://semver.org/).

## [Unreleased]

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
