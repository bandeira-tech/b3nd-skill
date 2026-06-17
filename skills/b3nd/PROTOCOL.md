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

## Base-path injection — letting deployers choose the mount point

A protocol module does not have to hard-code its scheme. It can export
a **factory** that takes a base path at construction time:

```
taskwatchProtocol("task://t/")                     // standalone
taskwatchProtocol("signed://0xabc.../taskwatch/")  // mounted inside a signed scheme
taskwatchProtocol("immutable://acme/tasks/")       // mounted inside an org-namespaced immutable surface
```

The factory builds programs keyed at `${basePath}<sub-prefix>` and URI
helpers prepend `basePath`. The shape, validators, classifiers, and
handlers are identical across mounts. The behavior **on top of** the
mount is whatever the outer scheme guarantees — which is how the
protocol inherits signing, immutability, or encryption from its host
scheme without having to re-implement those rules.

When designing this way:

- **URI helpers are the only public way to build URIs.** No caller
  should hard-code `task://t/<id>` — if they do, the mount-point
  abstraction leaks.
- **Validate the base path in the factory.** It must end with `/`,
  carry a scheme and host, and reject bases incompatible with the
  protocol's writes (e.g. mounting a mutable index under
  `hash://sha256/`).
- **Multi-tenancy falls out.** The base path is a tenant key — two
  instances of the same protocol on one rig under different bases
  share no state.
- **Discoverability is the cost.** `signed://0xabc.../taskwatch/` is
  less self-describing than `task://t/` in a route table. Consider
  exposing a manifest (`{ name, basePath, programKeys }`) so operators
  can introspect what's mounted where.

This is independent of the behavior-named-scheme guidance above.
Either composes without the other; together they give you protocol
modules that are reusable across trust models and namespaces.

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

Protocol authors do **not** ship transport or storage choices. Those
are operator concerns.

## What to verify in code before writing it

When the user asks you to design a protocol, do not generate import
statements or function calls until you have done the relay in
TARGETS.md. The shape of programs and handlers is stable; exact type
names and helper imports are not.
