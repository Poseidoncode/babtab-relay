# Changelog — @babtab/relay

All notable changes to the public npm package `@babtab/relay`
are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [Semantic Versioning](https://semver.org/).

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
