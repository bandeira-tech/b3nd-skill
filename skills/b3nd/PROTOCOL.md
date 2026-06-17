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

Protocol authors do **not** ship transport or storage choices. Those
are operator concerns.

## What to verify in code before writing it

When the user asks you to design a protocol, do not generate import
statements or function calls until you have done the relay in
TARGETS.md. The shape of programs and handlers is stable; exact type
names and helper imports are not.
