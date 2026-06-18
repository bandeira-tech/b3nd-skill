# Building an app on a protocol

App developers consume a protocol. The protocol gives you programs and
handlers; B3nd Core gives you a Rig and the `ProtocolInterfaceNode`
clients to wire it to nodes.

## Mental model

To build an app, you only need three operations on the Rig:

- **receive** — submit `Output[]` (each is `[uri, payload]`). The rig
  runs the protocol's program for that URI, dispatches to the matching
  handler, and broadcasts the handler's emissions through the configured
  receive routes.
- **read** — fetch state at one or more locators. Returns one
  `Output<T>` per input, positionally aligned.
- **observe** — stream `string[]` batches of URIs that fired. Re-read
  to learn the new state.

The fourth primitive, **status**, reports health and capabilities.

## A typical app boot

1. **Identity** — load or generate an Ed25519 keypair. Persist it in
   the browser, a wallet, or a managed key store.
2. **Clients** — instantiate the PIN clients that point at the nodes
   your deployment uses (one transport per upstream, mixed if needed).
3. **Rig** — construct a `Rig` with `routes` (which client serves which
   URI patterns), the protocol's `programs` and `handlers`, and any
   hooks/events/reactions you want.
4. **Subscribe** — call `observe` on the URI prefixes the UI cares
   about.
5. **Render** — read what the rig says is there; show it.
6. **Write** — when the user does something, build the tuple, sign or
   encrypt as the protocol requires, call `rig.receive`.

That's the loop. The protocol decides whether each write produces
persisted outputs.

## URIs for writes, locators for reads

When you call `receive`, you pass URIs — canonical identifiers. When
you call `read` or `observe`, you pass **locators** — opaque addressing
strings the upstream client interprets. A locator might be:

- A bare URI (the simple case).
- A URI with query directives the client understands.
- A pattern matching many URIs (for list-style reads).

The framework does not define locator grammar. The executing client
does, and the protocol documents what its operators support.

## Identity and signing

The protocol decides what a valid signer looks like. The app provides
one. Common patterns:

- Generate a key on first launch, store in `localStorage` or
  `IndexedDB`.
- Derive keys from a mnemonic the user can back up.
- Delegate to a browser wallet or external signer.

For pubkey-gated URIs, the app signs the payload (or an envelope around
it) before calling `receive`. The protocol's program verifies the
signature; the framework does not.

## Encryption

If the protocol uses encrypted URIs, the app encrypts before `receive`
and decrypts after `read`. Nodes never see cleartext. Symmetric data
shared between known parties typically uses keys derived from
participants' public keys; broadcast-but-private data uses a group key
distributed out of band.

## Transports

Apps pick transports based on environment:

- Browser → HTTP and/or WebSocket clients against a node URL.
- Deno / Node service → HTTP, WebSocket, gRPC-over-HTTP, MCP, or
  in-process clients.
- Tests → in-process / functional clients with no network.

A single Rig can mix transports per route. Reads can hit a CDN over
HTTP while receives go over WebSocket to an ingest node, served to your
own app over gRPC — all without touching protocol code.

## Errors

The Rig speaks one shape: `Output[]` for reads, `ReceiveResult[]` for
receives, exceptions for everything that's not the protocol's business:

- Network down, no route accepts the locator, malformed input → throws.
- Domain refusals, missing data, auth failures → live in the payload
  shape by the executing client's convention.

When something throws, it's a wire or wiring problem; when something
comes back as a domain-shaped response, it's the protocol talking.

## Local caching

If the app wants reads to work offline or with lower latency, wire a
local store (memory, localStorage, IndexedDB) as one of the read-route
clients. The Rig will broadcast receives to it alongside the remote
ingest route, and serve reads from it before falling back to remote.
Local stores share the PIN interface — no special code path.

## Before writing code

Do not import from `@bandeira-tech/*` packages without first running
the relay protocol in TARGETS.md. Package list, version, and export
shape live there.
