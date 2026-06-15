# Vision

B3nd is a framework for building DePIN protocols. It gives you URI-addressed
resources, schema-driven validation, and cryptographic primitives. You define
the rules — what data is allowed, who can write where, how messages are
validated — and B3nd handles dispatch, storage, transport, and encryption.

**Protocols are built on B3nd. Apps are built on protocols.**

## The three roles

B3nd serves three audiences at three layers of the stack. Knowing which one
the user is in determines what they need from you.

**Protocol designers** define the rules of a network. They write programs
(validation functions), choose URI conventions, and pick a trust model. The
framework gives them primitives; they assemble the rules. This is B3nd's
primary audience.

**App developers** build on top of an existing protocol. They use the SDK
plus a protocol's schema to write applications. They think in their domain
(recipes, invoices, profiles, posts) and call receive/read/list to interact
with the network. They don't need to understand framework internals.

**Infrastructure operators** run B3nd nodes loaded with a protocol's schema.
They pick backends, manage replication, handle uptime. They deliver the
mail; they don't read it.

When a user asks for help, figure out which role they are in before
proposing anything.

## The message tuple

Every state change in B3nd is a tuple of `[uri, values, data]`.

- **uri** addresses where the data goes. The URI scheme determines which
  program validates the write.
- **values** carry conserved quantities — things the protocol may need to
  reason about (amounts, counts, sequence numbers).
- **data** is the payload itself.

This is the universal primitive. Everything else — envelopes, signatures,
encryption — is wrapping around tuples.

## Programs

A program is a pure function that classifies a message. It receives a
message and returns a result: accept, reject, or some protocol-defined
outcome. Programs have no side effects. They are the protocol designer's
rulebook in code.

Programs do not store data. They decide whether a message is well-formed
under the rules of the protocol. Storage is a node-operator concern.

## Schemas

A schema is a map from URI prefixes to programs. The framework dispatches
each incoming message to a program by looking up its URI in the schema. A
protocol *is* its schema, plus whatever conventions it documents around
URI design.

## Envelopes

An envelope bundles inputs (URIs consumed) and outputs (new tuples) into a
single atomic-intent unit. Content-addressed envelopes give replay
protection and audit trails. Envelopes are how a protocol expresses "these
writes go together."

## Cryptography is client-side

Identity uses Ed25519. Confidentiality uses X25519. Content addressing uses
SHA-256. Deterministic key derivation uses HMAC. All of it happens in the
client, before the message reaches a node.

This means: **nodes are untrusted by design**. Privacy is achieved by
encrypting before sending. The same node happily serves public and private
data without configuration changes.

## Storage is an operator concern

The framework validates and dispatches. Whether an accepted message is
stored, cached, forwarded, or replicated is decided by the node operator.
App developers and protocol designers do not couple to a backend.

## Transport is uniform

Different transports (HTTP, WebSocket, in-process, gRPC) all implement the
same node interface. You can swap transports without changing protocol or
app code.

## What B3nd is not

- Not a blockchain. There is no global consensus, no chain, no fee market.
- Not a database. It does not pick a storage model for you.
- Not a CMS. It does not assume any particular kind of content.
- Not opinionated about identity. You bring the keys.

B3nd is the rails. Protocols are the routes. Apps are the trains.

## Where to go next

- Designing a protocol → PROTOCOL.md
- Building an app on a protocol → APP.md
- Running a node → OPERATOR.md
- Writing any code → TARGETS.md first
