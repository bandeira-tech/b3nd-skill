# Building an app on a protocol

App developers consume a protocol. The protocol gives you a schema and
(usually) typed helpers; B3nd gives you a client that speaks to nodes.

## Mental model

To build an app, you only need three operations:

- **receive** — submit a message tuple to the network. The node dispatches
  it through the schema. The program decides whether it's accepted.
- **read** — fetch the current value at a URI.
- **list / observe** — iterate or subscribe to a URI prefix.

Everything else (identity, encryption, transport) is composition around
these primitives.

## Identity

The user's identity is a keypair. The app holds the private key; the
public key shows up in URIs and signatures. There is no account system in
the framework — identity is just "can you sign with this key?"

Common UX patterns:

- Generate a key on first launch, store it in browser storage.
- Derive keys from a mnemonic the user can back up.
- Delegate to a wallet that holds the key.

The protocol decides what a valid signer looks like. The app provides one.

## Encryption

If the protocol uses encrypted URIs, the app encrypts before calling
receive and decrypts after read. Nodes never see cleartext. This is
deliberate: it means a node operator cannot be subpoenaed for content
they never had.

For symmetric data shared between known parties, derive a key from the
participants' public keys. For broadcast-but-private data, encrypt to a
group key the app distributes out of band.

## Transports

Apps pick a transport based on environment:

- Browser → typically HTTP or WebSocket against a node URL.
- Deno / Node service → HTTP, WebSocket, or in-process.
- Tests → in-process / memory, no network.

The same client interface works across all of them. Swapping is a one-line
change.

## Storage on the client

Apps often want a local cache so reads don't always hit the network. B3nd
provides client-side stores (in-memory, localStorage, IndexedDB) that
follow the same interface as a remote node. You can read from a local
store and replicate from a remote one without rewriting app code.

## What you actually write

A typical B3nd app is:

1. Boot — load or generate identity, pick a node URL, pick a transport.
2. Subscribe — listen to the URI prefixes the UI cares about.
3. Render — read from the local cache; show what the protocol says is
   there.
4. Write — when the user does something, build a tuple, sign it, call
   receive.

That's the loop. The protocol's schema decides whether each write
succeeds.

## Before writing code

Do not import from `@bandeira-tech/*` packages without first running the
relay protocol in TARGETS.md. The current package list and export shape
live there.
