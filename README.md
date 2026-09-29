# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Version](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

The **local relay server** for the Babtab Chrome extension: it connects your AI agent (Cursor / Pi / Claude Code / …) to your real Chrome.

MV3 extensions cannot listen on a port, so this tiny program acts as the bridge. It runs on `localhost`, so traffic never leaves your machine. The relay only forwards messages — **it cannot see your page content**.

## How it connects (3 roles)

```text
AI agent (Cursor / Pi …) ←→ Relay (local :3000) ←→ Chrome extension (with Side Panel)
   connects via MCP config     forwarding + pairing only    does the real work in your tabs
```

## Quick start (no clone, no install)

### Step 1: Install the Chrome extension

`chrome://extensions` → enable **Developer mode** → **Load unpacked** → select the `dist` folder.

> Once published on the Chrome Web Store, this step becomes "install from the store".

### Step 2: Start the relay (pick one, same result)

```bash
# A. Have Node 20+? No install needed, just run:
npx @babtab/relay

# B. No Node? Download the binary for your OS from GitHub Releases:
./babtab-relay-darwin-arm64   # e.g. macOS Apple Silicon
```

You're up when you see (default port `3000`):

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

Advanced (custom port / token file location):

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` doesn't install anything — it downloads and runs once. Closing the terminal stops the relay.

### Step 3: Connect the extension to the relay

1. Open any website in Chrome, click the Babtab toolbar icon to open the **Side Panel**
2. Relay URL is pre-filled with `ws://127.0.0.1:3000` — leave it
3. Click **"Save & Connect"**

This registers your Chrome as a device on the relay.

### Step 4: Connect your AI agent to the relay (Cursor as example)

1. On the same Side Panel page, pick your agent → click **"Copy config"**
2. Paste into `mcpServers` in `~/.cursor/mcp.json` (or the project's `.cursor/mcp.json` for single-project use):

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer the-token-you-just-got" }
    }
  }
}
```

3. When the panel shows `Controlled by: cursor`, you're connected.

One sentence to verify (ask your agent):

> Use browser_observe to look at the tabs open in my Chrome, then tell me their titles and URLs.

## FAQ

- **"Save & Connect" does nothing?** Check the relay terminal for `listening` first, then confirm the URL is `ws://127.0.0.1:3000` with a matching port.
- **Pairing code expired?** Codes are short-lived — just hit "Copy config" again.
- **Want a new token?** Re-running step 4 issues a new token; remember to update `mcp.json`.
- **Agent and Chrome on different machines?** (Advanced) Put the relay on a VPS and change the Side Panel's Relay URL to your `wss://…`. The flow stays the same.

## Privacy

- On `localhost` it's a purely local connection — packets never leave your computer.
- The relay only forwards commands and results. It never parses or stores page content.
- The Side Panel can **Pause / Take Over / Disconnect** at any time — the human always has final control.

## Developers

This repo only contains release artifacts (one obfuscated bundle + binaries), not the development source. Please file issues and discussions right here.

## License

Apache-2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE).

Copyright 2026 Poseidoncode.
