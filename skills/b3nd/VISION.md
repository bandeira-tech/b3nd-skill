# Vision

B3nd is a framework for systems that put a user's data on
addressable rails the user owns. It gives you URI-keyed state,
pure-function validation, and cryptographic primitives. You define
the rules; B3nd handles dispatch, storage, transport, and encryption.

**Protocols are built on B3nd. Apps are built on protocols.**

## The three roles

**Protocol designers** define the rules of a network — programs
that classify, handlers that transform, URI conventions, trust model.

**App developers** build on an existing protocol. They run a Rig
configured with the protocol's programs and handlers and call
receive/read/observe in their domain.

**Infrastructure operators** run B3nd nodes loaded with a protocol,
backed by storage, exposed over one or more transports.

Figure out which role the user is in before proposing anything.

## The message tuple

Every addressed piece of state is an `Output = [uri, payload]`. Two
slots: the URI says **where**, the payload says **what**. Both are
opaque to the framework — payload shape is the protocol's concern.

Protocols that need to reason about quantities (balances, counts,
sequence numbers) derive them from the payload inside their own
programs. B3nd does not carve out a separate "values" slot.

## Programs and code handlers

A protocol's rules live in two pure-function tables on the rig:

- **Programs** — `Record<uri-prefix, Program>`. Classify the
  `[uri, payload]` and return a protocol-defined code. Pure
  classifiers; they never store, mutate, or transmit.
- **Code handlers** — `Record<code, CodeHandler>`. Receive the
  classified output and return the `Output[]` for the rig to
  dispatch. Pure transforms: persist, decompose, refuse, conditionally
  emit.

The rig owns the wire. Handlers say what should happen; the rig
dispatches it.

A record doesn't have to be a single payload at a single URI. Many
protocols are cleaner as a tree of URIs per record, with state
derived by listing and folding. See PROTOCOL.md.

## The Rig

The harness that wires it together. A Rig is constructed with:

- **routes** — which client serves receive, read, and observe for
  which URI patterns.
- **programs** and **handlers** — the protocol's two tables.
- **hooks**, **events**, **reactions** — observation and side-effect
  attachment points.

The rig itself implements `ProtocolInterfaceNode` (PIN), so any rig
is a node: it can be served over any transport or composed as an
upstream of another rig.

## URIs vs locators

Writes are addressed by **URI** — the canonical identifier the
payload lives under. Reads and observes take **locators** — opaque
strings the executing client interprets. A locator can be a bare
URI, a URI plus directives, a pattern with wildcards, anything the
client's grammar accepts.

The scheme is the protocol's most expensive choice — it shapes how
programs classify and how operators wire routes. Prefer schemes that
name a *behavior* the program enforces over schemes that name a
*domain*. See PROTOCOL.md.

## Users own the data; apps agree on addressing

The default in most software is the opposite: each app owns its
backend, each backend owns its database, and the user's data is
whatever they can export through an API the vendor controls.

B3nd inverts it. A rig is data wiring, not application logic. A user
runs (or rents) a rig. Apps mount onto it under basepaths the user
picks. The rig stores, routes, authenticates, and encrypts; it does
not interpret.

Two consequences:

- **No moats on data.** A second app that follows the same
  addressing convention can read what the first app wrote. No
  integration contracts, no per-app APIs.
- **The user is the substrate.** The same app code, swapping
  basepaths and credentials, works on the user's rig, the org's, or
  a community's.

The primitives — message tuple, URI addressing, client-side
cryptography, pluggable storage — split this way so the rig holds
nothing app-specific. Interpretation lives in the protocol module
the apps share.

## Cryptography is client-side

Identity uses Ed25519. Confidentiality uses X25519. Content
addressing uses SHA-256. All of it happens before a message reaches
a node.

**Nodes are untrusted by design.** The same node serves public and
private data without configuration changes — privacy comes from
encrypting before sending.

## Storage is an operator concern

The framework validates and dispatches. Whether an accepted output
is persisted, cached, replicated, or forwarded is decided by the
operator when wiring routes. App and protocol code never couple to a
backend choice.

## Transports are uniform

HTTP, WebSocket, gRPC-over-HTTP, MCP, and in-process all implement
the same `ProtocolInterfaceNode`. One rig can serve one transport
while its routes are backed by clients on entirely different
transports.

## Errors

The framework speaks one shape end-to-end: `Output[]`. No separate
error channel.

- Transport and programmer errors (network down, no route accepts,
  malformed locator) **throw**.
- Everything else — domain refusals, auth failures, missing data —
  lives in the payload by convention defined by the executing client.

## What B3nd is not

- Not a blockchain. No global consensus, no chain, no fee market.
- Not a database. It does not pick a storage model for you.
- Not a CMS. It does not assume any kind of content.
- Not opinionated about identity. You bring the keys.

B3nd is the rails. Protocols are the routes. Apps are the trains.

## Where to go next

- Designing a protocol → PROTOCOL.md
- Building an app on a protocol → APP.md
- Running a node → OPERATOR.md
- Writing any code → TARGETS.md first
