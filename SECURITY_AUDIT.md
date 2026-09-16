# Security Audit

Date: 2026-09-16

## Verdict

**Clean.** This is a genuinely minimal Manifest V3 browser extension. Total code
surface is ~40 lines across `background.js` and `rules.json`; there's nothing
else to it.

## Remote code execution

None possible. There is no `eval`, no `new Function`, no dynamic `<script>`
injection, no `fetch`/`XMLHttpRequest` anywhere in the code. MV3 also
structurally forbids remotely-hosted code for extensions, and this one
doesn't attempt any.

## Phone-home / tracking

None. There is zero networking code - no `fetch`, `XMLHttpRequest`,
`WebSocket`, `sendBeacon`, or analytics SDK anywhere in the extension. The
only "network" behavior is the declarative redirect rules (`rules.json`),
which are pattern-matched and applied by the browser itself, not by
extension code reaching out anywhere. `PRIVACY.md`'s claim of "no
tracking/cookies/analytics/3rd party services" checks out against the actual
code.

## Permissions (manifest.json)

- `declarativeNetRequest` + `storage` - both minimal and appropriate for what
  it does.
- `host_permissions` limited to `*.zoom.us` and `*.zoomgov.com` - not a broad
  `<all_urls>` grab.
- No `scripting`, `webRequest`, `tabs`, `cookies`, or other sensitive
  permissions.

## What the code actually does

- `background.js`: toggles a badge and enables/disables a
  declarativeNetRequest ruleset on icon click, storing one boolean
  (`disabled`) in `chrome.storage.local`. That's the entire logic.
- `rules.json`: four static regex-based redirect rules that rewrite
  `zoom.us/j/<id>` and `/s/<id>` (and the zoomgov equivalents) to the
  `/wc/.../join` or `/wc/.../start` web-client URLs. All redirects stay
  within the zoom.us/zoomgov.com domains - it can't redirect you off to an
  attacker-controlled host.

## Supply chain risk

None. There is no `package.json`, no `node_modules`, no build step, no
bundler, and no third-party library of any kind (not even a small utility
lib) - just hand-written vanilla JS and a static JSON ruleset. Nothing to
compromise upstream.

## Notes

This is `arkadiyt`'s open-source "Xoom Redirector" (renamed after a Zoom
trademark dispute, per the README), currently at v1.0.5 with a clean git
history. Nothing in the audit suggests any deviation from its stated
purpose.
