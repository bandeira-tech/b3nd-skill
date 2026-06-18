# Designing a protocol

A protocol on B3nd is three things together:

1. A **URI scheme** — the address space your protocol owns.
2. A set of **programs** — URI-prefix → classifier function returning
   a protocol-defined code.
3. A set of **code handlers** — code → pure transform that returns the
   `Output[]` the rig should dispatch.

You design these three; the framework dispatches and persists.

## Start from the messages

Before writing code, write down the messages your protocol needs. Each
message is `[uri, payload]`. For each one, ask:

- What is the URI? What does the prefix say about who owns it?
- What does the payload carry? Is it cleartext, encrypted, hashed,
  signed?
- What rule decides whether this message is valid?
- Which classification code(s) can the program return for this URI?
- What should happen for each code — persist, decompose, refuse, emit
  follow-up tuples?

If you can describe every message that way, you can write the protocol
as a `{ programs, handlers }` pair.

## URI conventions

URIs express *behavior*, not meaning. The prefix selects the program;
the suffix is whatever the protocol wants. Common patterns:

- **Owned by a pubkey** — the URI embeds the writer's key. The program
  verifies the signature.
- **Content-addressed** — the URI is the hash of the payload. The
  program checks the hash.
- **Immutable** — the program returns a refusal code if a value already
  exists at the URI; the matching handler returns `[]`.
- **Encrypted** — the URI advertises that payload is ciphertext; the
  program never tries to interpret it.

Pick conventions that match your trust model. Document them in the
protocol's README. Operators will use them to wire routes.

**Don't put an opaque id directly after the scheme.** A URI like
`thing://abc123` puts `abc123` in the URI's *host* slot — the part
program prefixes match on. A program keyed at `thing://abc123` wouldn't
generalize; a program keyed at `thing://` has no further structure to
classify against. Insert at least one stable path segment so the
routable prefix is meaningful: `thing://t/abc123`, `thing://by-id/abc123`,
etc. This also keeps related locators (indexes, sub-resources) inside
the same routable prefix as the items themselves — e.g. `thing://t/list`
routes alongside `thing://t/<id>`.

## Schemes name behavior; paths name the application

"URIs express behavior, not meaning" has a sharper edge worth naming.

When you pick a scheme, prefer one that names **what the program
enforces** over one that names **what the data is about**. Schemes like
`hash://sha256/`, `immutable://`, `signed://<pubkey>/`,
`mutable://<pubkey>/`, `encrypted://` describe a *rule*. Schemes like
`task://`, `info://`, `recipe://` describe a *domain*.

Both work. The behavior-named approach has two payoffs:

- **Composition.** Multiple apps can share a foundational scheme. A
  "decisions log" and a "task index" both want immutability — they can
  both write under `immutable://` (in different path namespaces) and
  inherit the same program.
- **Operator clarity.** Operators wire backends to *behaviors* —
  "immutable goes to object-locked S3; mutable goes to Postgres" —
  rather than per-app schemes that re-decide the same questions.

Domain-named schemes are not wrong. They're one app's worth of
foundational surface written from scratch. If your protocol is
genuinely sui generis, invent a new scheme. Otherwise, prefer to
compose.

See also: base-path injection below for letting a single protocol
module mount under different schemes at deploy time.

## Decompose records into URI subtrees

A record doesn't have to be a single JSON payload at a single URI.
Often the cheapest protocol shape is a **tree of URIs** under a
per-record root, where each fact is its own URI and current state is
derived by listing and folding — not by parsing a document.

A typical layout under the record root uses a few named subspaces:

```
<root>/data/<field>          # current value of each data field
<root>/meta/<field>          # bookkeeping (createdAt, owner, version, ...)
<root>/entries/{ts}-{kind}   # append-only history; kind in the URI, body in the payload
```

Protocols add more subspaces as they need them (`tags/`, `resources/`,
`context/`, ...). The convention to hold to:

- **`data/`** — the canonical "what this record holds right now."
  Last-write-wins per field. The agent updates `data/title` by writing
  that URI; no record-rewrite needed. Don't put data fields like
  `title` or `description` directly at the record root — keep them
  inside `data/` so the root is a namespace, not a record body.
- **`meta/`** — protocol-defined bookkeeping. Same shape as `data/`,
  separated by concern.
- **`entries/`** — append-only history. The entry kind is in the URI
  (`{ts}-status-paused`, `{ts}-progress`, `{ts}-note`); the payload is
  the human-readable body. Status, last-updated-at, count are all
  derivable from the URI list alone — payloads are optional for
  derivation.

What this kills:

- Read-modify-write boilerplate. Appending an event is one write.
- Full-record rewrites on every change. Editing one field is one
  write to that field's URI.
- Schema mismatch on partial updates. The shape is the URI tree, not
  a record schema.
- Hot-row contention. Concurrent writers updating different fields
  touch different URIs.

What it gives up:

- **Single-document atomicity.** A multi-URI write is multiple
  operations. Order writes so a partial failure is recoverable (e.g.
  write `data/title` last so a half-failed create looks "untitled"
  and is fixable).
- **Record-shape validation.** Programs validate URI shapes
  (`entries/` leaves must be `{ts}-{kind}`, tags must match a
  charset, etc.), but they can't reject "you forgot to write a title"
  because there is no record to reject. Missing fields render as
  empty values by convention.

**Enumeration.** Foundational stores (FS, plain S3) may not list
across path depths. Maintain an explicit `<basepath>index/<sortable-key>`
URI as a cheap secondary index, treated as cache-like derived state
rebuildable from the source.

**Derived views.** Expose a synthetic locator (e.g. `<root>?fn=view`)
that the protocol's node resolves by issuing the component reads in
parallel and returning the folded state in one call. The underlying
URIs remain canonical; `?fn=view` is convenience.

## Mount basepaths so users keep control of their data

A protocol module does not need to hard-code where it lives. Export
a **factory** that takes a basepath at construction time:

```
taskwatchProtocol("taskwatch://")
taskwatchProtocol("signed://0xabc.../taskwatch/")
taskwatchProtocol("acme/internal/tasks/")
```

The shape, programs, handlers, and code vocabulary are identical
across mounts. What changes is **where** on a user's rig the protocol
lives.

This is in service of the bigger point (see VISION.md): B3nd is built
so users own the data and apps agree on addressing. A protocol that
hard-codes its scheme makes the app the moat — it dictates where its
data lives, and other apps can't see or extend it. Letting deployers
pick the mount inverts that. The rig is data wiring the user
controls; apps commit to the addressing convention and ride a surface
the user already runs.

Practical guidance for the factory:

- **URI helpers are the only public way to build URIs.** Hard-coded
  locators leak the mount-point abstraction.
- **Validate the basepath in the factory.** End it with `/`, require
  a scheme separator, reject bases incompatible with the protocol's
  writes.
- **Multi-tenancy falls out.** Two mounts under different basepaths
  are independent state on the same rig.
- **Treat the storage root as a separate constructor argument.** The
  URI basepath says where the protocol lives on the wire; the
  storage root says where bytes land. Operators vary them
  independently.

The behaviour layered on top of a mount is whatever the outer scheme
guarantees — mounting under `signed://<pubkey>/` inherits signature
gating; under `immutable://`, write-once. The protocol gains those
rules for free instead of re-implementing them.

## status() is the protocol's manifest

The rig already returns a `StatusResult` from `status()`. Hydrate
it. A protocol mounted on a rig should report:

- Its schema(s).
- Its code vocabulary (`ok:*`, `refuse:*`).
- A handful of URI examples (point reads, listing locators).
- Mount metadata (basepath, store kind).

This is the protocol's runtime self-description. Skills, MCP tool
descriptions, app UIs, catalog consumers all read from one place
instead of duplicating the protocol's shape across configs.

Users browsing their own rig see what is mounted on their data.
Apps connecting to a rig discover the addressing and trust
conventions to follow. The status payload is the contract the rig
publishes for everything riding on it.

## Use protocols here to push-test canon

The patterns above (behavior-named schemes, base-path injection,
operator-wired backend behaviors) are the shape `b3nd-canon` is
heading toward — canonical `immutable://`, `signed://`, `mutable://`,
`encrypted://` protocols that everyone composes against instead of
re-inventing.

Canon is not there yet (see TARGETS.md). Until it is, every protocol
you write following these patterns is a **probe** for what canon
should ship. When you implement immutability in a handler, treat that
handler as canon's draft for `immutable://`. When you implement
signature-gating, that's canon's draft for `signed://`. Keep the
implementation small enough that it could be lifted into canon
without a rewrite.

Surface what surprised you — the validation rule that wasn't obvious,
the edge case the URI grammar didn't cover, the operator-wiring story
that needed bespoke glue. Those surprises are the spec canon needs.

## Programs as classifiers

A program is a pure async function. Inputs: the `Output` being
classified, its upstream parent (or `undefined` at the top level), and
a `read` function for confirmed-state lookups. Output: a
`ProgramResult` carrying a code.

Programs do **not**:

- Store data, mutate global state, or make network calls.
- Decide what to dispatch — that's the handler's job.
- Sub-classify by calling back into the rig. If a program needs to
  break a payload into sub-outputs, it does so internally and returns
  a code that the matching handler decomposes.

## Code handlers as transforms

A code handler is a pure async function. Inputs: the classified
`Output`, the `ProgramResult`, and a `read` function. Output: the
`Output[]` to dispatch through `routes.receive`. Common shapes:

- **persist** — return `[out]`. The simple "this is valid, store it"
  case.
- **decompose** — return `[envelope, ...payload.outputs, ...deletions]`.
  An envelope-style write that produces multiple tuples.
- **conditional** — read existing state via `read`, return `[]` to
  refuse or `[out]` to accept.
- **refuse** — return `[]`. Classification said no.

Handlers run after the program classifies. Handler emissions skip
re-classification (handlers are canonical interpreters). Reactions run
after broadcast lands.

## Choosing a trust model

B3nd does not pick a trust model for you. Common ones:

- **Open** — anyone can write anywhere; programs validate form only.
- **Pubkey-gated** — writes are accepted only if signed by the key the
  URI identifies.
- **Managed** — a single operator (or quorum) signs off on writes.
- **Conservation-based** — handlers enforce that inputs consumed equal
  outputs produced.

The model lives across programs (classify) and handlers (emit). You can
mix models in one schema — different URI prefixes can use different
trust postures.

## Packaging a protocol

A protocol is shipped as a module that exports `{ programs, handlers }`
(and any helpers for app developers — URI builders, typed payload
helpers, etc.). Apps and operators both depend on the same module: they
construct a `Rig` with those tables and wire routes appropriate to the
deployment.

Ship the module so it runs on **both sides** of every transport. The
same `protocol.ts` should import cleanly into a Deno server, a
browser bundle, an MCP boot script. The program runs locally wherever
it imports — a browser app refusing an invalid write does so with
the same refusal code the server would emit, without a round trip.
The module is the protocol's contract; transports just carry the
tuples.

Protocol authors do **not** ship transport or storage choices. Those
are operator concerns.

## What to verify in code before writing it

When the user asks you to design a protocol, do not generate import
statements or function calls until you have done the relay in
TARGETS.md. The shape of programs and handlers is stable; exact type
names and helper imports are not.
