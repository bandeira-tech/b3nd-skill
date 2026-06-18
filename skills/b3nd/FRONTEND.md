---
level: L2
audience: browser-app developers building on a B3nd protocol
assumes: familiarity with a modern frontend framework (React/Vue/Svelte/Signals) and bundlers; APP.md absorbed
---

# Building browser apps over B3nd

A B3nd protocol can run in the browser by importing the same
`protocol.ts` the server uses and reaching the rig over HTTP or
WebSocket via a PIN client.

This file teaches the patterns. For current API surface (package
names, browser entry points, store/transport classes), follow the
relay protocol in TARGETS.md.

## One protocol module, two consumers

The protocol module (programs + handlers + URI helpers + codes)
imports cleanly into a browser bundle. The browser app's PIN wrapper
runs the program **locally** before calling `client.receive(...)`.
Three consequences worth naming:

- Refusal codes surface in the UI without a round trip.
- The vocabulary is uniform — server and browser classify the same
  output the same way.
- Validation lives in one place. Adding a new code or changing a
  refusal predicate touches one file; both sides update together.

When the user wants the form to refuse a write the same way the
server would, this is how — not a duplicated client-side validator.

## Reads

Reads go through a transport client (HTTP GET / WebSocket) that
implements the PIN interface. The URI helpers from the protocol
module construct locators; the client returns `[uri, payload]`
tuples.

For listing locators (`<base>/?fn=ls&format=...`), the protocol
decides what comes back — URIs only, URIs with payloads, counts. The
browser receives whatever the protocol promised; no transport-
specific shape coercion.

## Live state via observe

The PIN `observe` primitive maps to a WebSocket or NDJSON stream
from the rig. Subscribe to a URI pattern (e.g. `<basepath>**`) and
re-render the affected pieces of state when matching tuples arrive.

A URI-subtree record layout (PROTOCOL.md: "Decompose records into
URI subtrees") gives natural per-fact granularity for invalidation —
the observe stream tells you which URI changed, and the URI tells
you which piece of UI to update without a full record reload.

## Bytes on the wire

`hash://` reads (and other byte-valued URIs) come back as raw bytes.
The HTTP/WS transport returns them as `Uint8Array`; the browser
decides how to decode (UTF-8 to string, base64, blob, ...).

Don't accept JSON-serialized byte objects (`{ "0": 66, "1": 117, ... }`)
from the wire — that's a node-layer bug to fix upstream, not a
client concern to work around. If you see it, the rig's read path
isn't bridging bytes correctly.

## Reactivity

If the app needs reactive state (React/Vue/Svelte/Signals), wrap
the rig's PIN surface in a small store layer that:

1. Maintains a cache keyed by URI (or by listing locator).
2. Subscribes to `observe` streams for the URI patterns the UI
   cares about.
3. Re-emits cache changes through the framework's reactivity
   primitive.

The B3nd packages may ship browser-friendly stores out of the box;
check TARGETS.md.

## status() as a UI config source

The same `b3nd_status` payload that drives MCP tool discovery can
drive the browser app's static UI surfaces — filter dropdown
vocabularies, schema-aware form fields, code-to-message lookups for
refusal errors. Read it on app boot; cache it; re-read on protocol
upgrades. The protocol's runtime self-description becomes the UI's
config without a separate config file.

## App shell vs rig embed

Two common deployment shapes:

- **Browser-as-client.** The rig lives on a server (HTTP node); the
  browser talks to it. Public Firecat nodes, hosted backends.
- **Browser-as-rig.** The rig runs in the browser tab, backed by a
  browser store (`localStorage`, `IndexedDB`). The same PIN surface;
  no server needed. Useful for offline-first / single-user apps.

Both shapes use the same protocol module. The choice is a routes-
configuration concern, not a code-shape one.

## Where to look next

- Designing the protocol — PROTOCOL.md
- App-side patterns shared with MCP servers — APP.md
- MCP-server specifics — MCP.md
- Current API symbols — TARGETS.md
