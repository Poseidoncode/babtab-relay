# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Version](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

> **Requires the Babtab extension:** [Install Babtab from the Chrome Web Store](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)

The **local relay server** for the [Babtab Chrome extension](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb): it connects your AI agent (Cursor / Pi / Claude Code / …) to your real Chrome.

MV3 extensions cannot listen on a port, so this tiny program acts as the bridge. It runs on `localhost`, so traffic never leaves your machine. The relay forwards requests and results in memory, including page observations and screenshots; it does not persist page content.

## Quick start (0.3.1)

Requires Node.js 20+ and the matching [Babtab extension](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb). Install the extension from the [**Chrome Web Store**](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb); Developer mode is for local development only.

1. Open Babtab in Chrome and click **Add to Cursor** (or **Add to VS Code** / **VS Code Insiders** / **Add to Hermes** — all one-click).
2. Confirm Add / Enable in your AI tool. It starts the local bridge automatically. **Add to Codex** / **Add to Antigravity** copy a one-step install snippet instead (Codex: paste in a terminal — the desktop app shares the CLI config; Antigravity: paste into mcp_config.json and restart the IDE).
3. Return to Babtab and **Approve** the first connection.
4. On a regular website, ask your agent: **Use babtab to observe the current tab**. Approve site access in Babtab if asked.

No separate relay command or terminal window is needed. Keep Chrome open.
The first npm download requires internet access. Pairing credentials are stored in `~/.babtab/`, independent of your working directory.

### Other AI tools

Open **Other AI tools / install with a command**, choose your tool and copy its personalized command:

```bash
npx -y @babtab/relay@0.3.1 setup --target windsurf --port 3000 --device <device-id-from-extension>
```

Run once, enable / reload MCP in your AI tool, then approve in Chrome. Setup preserves other servers and backs up existing config files. Supported automatic targets: Cursor, Claude Code, Windsurf, Copilot, Codex, Claude Desktop, Antigravity, Devin, Kimi and Hermes. Pi / Manual use the advanced HTTP flow.

### How automatic startup works

Your MCP client launches `babtab-relay stdio --device … --client … --port 3000`. The bridge starts or reuses the loopback relay. Tool discovery works before approval; browser operations require an explicitly approved token. Multiple clients share the relay; a remaining client takes over if its owner exits. In-flight actions interrupted by shutdown return an error and are never replayed automatically.

Credentials live in `~/.babtab/` (override with `BABTAB_DATA_DIR`). The extension reconnects after browser / MCP restarts. Explicit **Disconnect** stops automatic reconnection until you reconnect.

### Upgrading from 0.1.x

Stop the manually started relay, update the extension and add your AI tool again. Approve once to pair with the new managed credentials. Port conflicts or an older relay are reported without sending credentials to the conflicting service. Choose a different port under **Advanced connection settings** and reinstall the MCP entry if needed.

### Advanced: manual or remote relay

`npx @babtab/relay` still starts a standalone HTTP relay. In the extension, open **Advanced connection settings**, set the port / URL and use **Save & Connect**. Manual HTTP config generation remains available.

Standalone token storage uses `BABTAB_TOKEN_FILE` or `./data/relay-tokens.json` relative to the launch directory. Remote relays need an explicit `HOST` and TLS (`wss://` / `https://`). Use `setup --target <tool> --relay-url https://… --device <id>` after the relay and extension are connected.

### Troubleshooting

- **Cursor did not open?** Use the setup command under the button, then reload / enable MCP.
- **No pairing request?** Check that Node 20+ / `npx` is on your AI tool's PATH and Babtab is enabled. Keep the Chrome panel open. The MCP server's stderr logs explain startup problems.
- **Pairing expired?** Keep the AI tool running; it requests a new code automatically.
- **Connection stopped?** Reopen your AI tool and Chrome. If you explicitly disconnected, press Reconnect in Babtab.
- **Lost / revoked credentials?** Approve the new pairing request; the bridge never bypasses browser approval.

## Privacy

- On `localhost` it's a purely local connection — packets never leave your computer.
- The relay only forwards commands and results. It never parses or stores page content.
- The Side Panel can **Pause / Take Over / Disconnect** at any time — the human always has final control.

## Developers

This repo only contains release artifacts (one obfuscated bundle + binaries), not the development source. Please file issues and discussions right here.
