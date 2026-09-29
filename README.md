# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Version](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

The **local relay server** for the Babtab Chrome extension: it connects your AI agent (Cursor / Pi / Claude Code / …) to your real Chrome.

MV3 extensions cannot listen on a port, so this tiny program acts as the bridge. It runs on `localhost`, so traffic never leaves your machine. The relay forwards requests and results in memory, including page observations and screenshots; it does not persist page content.

## Upgrading to authenticated device connections

Update the relay and extension together, reload the extension, reconnect in the
Side Panel, and run Step 2 again for each agent. The extension creates a new
`dev_v2_` device ID; tokens paired to older IDs do not control the new connection.
The relay now binds `127.0.0.1` explicitly. A remote deployment must opt in with
`HOST` and use TLS (`wss://` / `https://`) to protect credentials in transit.

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
2. Step 1: confirm the port (default `3000`, must match the relay) — no URL typing needed
3. Click **"Save & Connect"**

This registers your Chrome as a device on the relay.
(Remote relay on a VPS? Use *Advanced: custom Relay URL* in the same step.)

### Step 4: Connect your AI agent to the relay (Cursor as example)

1. On the same Side Panel page, Step 2 → click **Cursor** → copy the one-line command:

```bash
npx -y @babtab/relay setup --target cursor --port 3000 --device <your-device-id>
```

2. Paste it into any terminal and run it. It shows a 6-digit pairing code —
   approve it on the banner at the top of the Side Panel.
3. Done: the command writes `babtab` into `~/.cursor/mcp.json` for you
   (existing entries preserved, previous file kept as `.bak`). Reload MCP in
   Cursor (Settings → MCP) and the panel shows `Controlled by: cursor`.

One sentence to verify (ask your agent):

> Use browser_observe to look at the tabs open in my Chrome, then tell me their titles and URLs.

Supported `--target` values: `cursor`, `claude-code`, `windsurf`, `copilot`,
`copilot-insiders`, `codex`, `pi`, `claude-desktop`, `antigravity`, `devin`,
`kimi`, `hermes`, `manual`. Run `npx -y @babtab/relay setup --help` for all
commands (`--list-targets` prints one ready line per harness). No terminal at
hand? Step 2 also offers *Pair & copy JSON manually* as a fallback.

## FAQ

- **"Save & Connect" does nothing?** Check the relay terminal for `listening` first, then confirm the Step 1 port matches the relay's port.
- **Pairing code expired?** Codes are short-lived — just re-run the setup command.
- **Want a new token?** Re-running the setup command issues a new token and rewrites the config (old `.bak` kept).
- **Agent and Chrome on different machines?** (Advanced) Put the relay on a VPS, use *Advanced: custom Relay URL* in Side Panel Step 1 with your `wss://…`, and pass `--relay-url https://…` to the setup command. The flow stays the same.

## Privacy

- On `localhost` it's a purely local connection — packets never leave your computer.
- The relay only forwards commands and results. It never parses or stores page content.
- The Side Panel can **Pause / Take Over / Disconnect** at any time — the human always has final control.

## Developers

This repo only contains release artifacts (one obfuscated bundle + binaries), not the development source. Please file issues and discussions right here.
