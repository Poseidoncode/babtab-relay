# Security Policy — @babtab/relay

Local-first relay. No hosted service, no telemetry, no account.

## Supported versions

| Version | Supported |
|---|---|
| 0.1.x | Yes |

Older versions: upgrade via `npx @babtab/relay@latest`.

## What the relay sees (and never sees)

The relay routes **metadata only**: device/client IDs, method names
(`browser_observe`, `browser_click`, …), pairing codes, auth tokens.

It **never** receives, stores, or logs:

- page DOM, screenshots, or extracted page content,
- browser credentials, cookies, or API keys belonging to other tools,
- fill/select input values (the extension redacts them before display,
  and the relay never persists request bodies).

Token store (`BABTAB_TOKEN_FILE`, default `./data/relay-tokens.json`)
holds random bearer tokens + scope + expiry only. Treat that file like
a password: `chmod 600`, never commit it, never send it to anyone.

## Running it safely

- Bind is localhost by default (`http://127.0.0.1:3000`). Do not expose
  it to a LAN / public IP unless you understand that anyone who can
  reach the port can attempt pairing.
- Pairing codes are 6-digit, single-use, and expire quickly. Approve
  only codes you just requested; reject anything you don't recognize.
- HIGH-risk browser actions (submit, pay, delete, publish, password
  fill) always require explicit approval in the extension Side Panel —
  the relay cannot bypass that gate.
- Prefer `npx @babtab/relay` (no install) or the GitHub Release binary
  over cloning the monorepo.

## Reporting a vulnerability

**Do not open a public issue for a suspected vulnerability.**

Email the maintainer or open a
[private security advisory](https://github.com/Poseidoncode/babtab-relay/security/advisories/new)
with:

1. affected version (`npx @babtab/relay --version` or tarball hash),
2. minimal reproduction steps,
3. impact assessment (what an attacker could / could not obtain).

We aim to acknowledge within 72 hours and will credit reporters
unless anonymity is requested.
