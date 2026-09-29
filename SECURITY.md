# Security Policy — @babtab/relay

Local-first relay. No hosted service, no telemetry, no account.

## Supported versions

| Version | Supported |
|---|---|
| 0.1.x | Yes |

Older versions: upgrade via `npx @babtab/relay@latest`.

## What the relay sees

The relay forwards request parameters and results in memory, including entered
values, page observations and screenshots. It does not persist page content or
log request/response bodies. It must be trusted by the extension and the agent.

The token store (`BABTAB_TOKEN_FILE`, default `./data/relay-tokens.json`) contains
bearer tokens, client names, device IDs, scopes and expiry. Device credentials
stay in extension local storage and are verified from the first WebSocket frame;
the relay does not persist them. Use TLS for every non-loopback deployment.

## Device authentication upgrade

Update both relay and extension, then re-pair agents using the new `dev_v2_`
device ID. Knowledge of the public ID alone no longer permits registration or
pairing approval. Old unauthenticated device connections are rejected.

## Running it safely

- Bind is localhost by default (`http://127.0.0.1:3000`). Do not expose
  it to a LAN / public IP unless you understand that anyone who can
  reach the port can attempt pairing.
- Pairing codes are 6-digit and expire quickly. Approve
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
