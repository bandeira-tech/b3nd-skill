# Vision

B3nd is a framework for building DePIN protocols. It gives you URI-addressed
resources, pure-function validation, and cryptographic primitives. You define
the rules — what data is allowed, who can write where, what happens on each
classification code — and B3nd handles dispatch, storage, transport, and
encryption.

**Protocols are built on B3nd. Apps are built on protocols.**

## The three roles

B3nd serves three audiences at three layers of the stack. Knowing which one
the user is in determines what they need from you.

**Protocol designers** define the rules of a network. They write programs
(classifiers that return codes), write code handlers (what to do for each
code), choose URI conventions, and pick a trust model. This is B3nd's
primary audience.

**App developers** build on top of an existing protocol. They run a Rig
configured with the protocol's programs and handlers, plus whatever
upstream nodes the deployment uses. They call receive/read/observe and
think in their domain (recipes, invoices, profiles, posts).

**Infrastructure operators** run B3nd nodes loaded with a protocol's
programs and handlers, backed by storage and exposed over one or more
transports. They pick backends, manage replication, handle uptime.

Figure out which role the user is in before proposing anything.

## The message tuple

Every addressed piece of state in B3nd is an `Output = [uri, payload]`.
That is the universal primitive. Two slots: the URI says **where**, the
payload says **what**. Both halves are opaque to the framework — payload
shape is the protocol's concern.

If a protocol needs to reason about quantities (balances, counts,
sequence numbers), it derives them from the payload inside its own
programs and handlers. B3nd does not carve out a separate "values" slot.

## Programs and code handlers

The protocol's rules live in two pure-function tables on the rig:

- **Programs** — `Record<uri-prefix, Program>`. A Program receives the
  `Output`, optional upstream context, and a read function, and returns a
  protocol-defined classification code (a `ProgramResult`). Programs are
  classifiers. They do not store, mutate, or transmit anything.

- **Code handlers** — `Record<code, CodeHandler>`. A CodeHandler receives
  the classified output, the program's result, and a read function, and
  returns the `Output[]` it wants the rig to dispatch. Handlers are pure
  transforms: persist, decompose into sub-tuples, refuse (return `[]`),
  conditionally emit.

The rig owns the wire. Handlers describe what should happen; the rig
dispatches it. Both layers are pure. A protocol is just programs +
handlers + URI conventions.

## The Rig

The rig is the harness that wires it all together. A protocol designer
or app developer constructs a `Rig` with:

- **routes** — which client(s) serve receive, read, and observe for
  which URI patterns. A route is a `connection(client, [patterns])`.
- **programs** — the URI-prefix → Program map.
- **handlers** — the code → CodeHandler map.
- **hooks**, **events**, **reactions** — observation and side-effect
  attachment points the framework provides.

The rig itself implements `ProtocolInterfaceNode` (PIN), so any rig is a
node: it can be served over any transport, or composed as an upstream of
another rig.

## URIs vs locators

Writes are addressed by **URI** — the canonical identifier the payload
lives under. Reads and observes take **locators** — opaque strings the
executing client interprets. A locator can be a bare URI, a URI plus
request-time directives, a pattern with wildcards, or anything else the
client's grammar accepts. The framework does not constrain locator
syntax; pattern-matching on routes uses a separate segment-glob grammar.

The scheme is the protocol's most expensive choice — it shapes how
programs classify and how operators wire routes. Prefer schemes that
name a *behavior* the program enforces over schemes that name a
*domain*. See PROTOCOL.md.

## Cryptography is client-side

Identity uses Ed25519. Confidentiality uses X25519. Content addressing
uses SHA-256. Deterministic key derivation uses HMAC. All of it happens
before a message reaches a node.

This means: **nodes are untrusted by design**. The same node serves
public and private data without configuration changes — privacy comes
from encrypting before sending.

## Storage is an operator concern

The framework validates and dispatches. Whether an accepted output is
persisted, cached, replicated, or forwarded is decided by the operator
when wiring routes to backends. App and protocol code never couple to a
backend choice.

## Transports are uniform

HTTP, WebSocket, gRPC-over-HTTP, MCP, and in-process all implement the
same `ProtocolInterfaceNode`. A single rig can serve one transport while
its routes are backed by clients on entirely different transports.

## Errors

The framework speaks one shape end-to-end: `Output[]`. There is no
separate error channel:

- Transport and programmer errors (network down, no route accepts,
  malformed locator) **throw** as exceptions.
- Everything else — domain refusals, auth failures, missing data —
  lives in the payload by convention defined by the executing client.

## What B3nd is not

- Not a blockchain. No global consensus, no chain, no fee market.
- Not a database. It does not pick a storage model for you.
- Not a CMS. It does not assume any particular kind of content.
- Not opinionated about identity. You bring the keys.

B3nd is the rails. Protocols are the routes. Apps are the trains.

## Where to go next

- Designing a protocol → PROTOCOL.md
- Building an app on a protocol → APP.md
- Running a node → OPERATOR.md
- Writing any code → TARGETS.md first
