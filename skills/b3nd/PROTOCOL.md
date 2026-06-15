# Designing a protocol

A protocol on B3nd is three things together:

1. A **URI scheme** — the address space your protocol owns.
2. A **schema** — a map from URI prefixes to programs.
3. A **trust model** — who can write where, and how that's enforced.

You design these three; the framework handles dispatch, storage,
encryption, and transport.

## Start from the messages

Before writing any code, write down the messages your protocol needs.
Each message is a tuple. For each one, ask:

- What is the URI? What does the prefix say about who owns it?
- What goes in values vs data? Values are for conserved quantities the
  protocol must reason about; data is the payload.
- What rule decides whether this message is valid?
- Who is allowed to write it?

If you can describe every message this way, you can build the protocol.

## URI conventions

URIs express *behavior*, not meaning. The prefix tells the framework which
program to dispatch to; the suffix is whatever the protocol wants it to
mean. Common patterns:

- **Owned by a pubkey** — the URI embeds the writer's public key. The
  program checks that the signature matches.
- **Content-addressed** — the URI is the hash of the data. The program
  checks the hash.
- **Immutable** — the program rejects any write to a URI that already has
  a value.
- **Encrypted** — the URI signals that data is ciphertext; the protocol
  doesn't try to interpret payload.

Pick the conventions that match your trust model. Document them in the
protocol's README.

## Programs as classifiers

A program is a pure function. Inputs: the message and whatever context the
framework provides. Output: a result that says accept or reject (and may
carry protocol-defined codes).

Programs do **not**:

- Store data.
- Mutate global state.
- Make network calls.
- Decide where the data lives.

They classify. That's it.

## Choosing a trust model

B3nd does not pick a trust model for you. Common ones:

- **Open** — anyone can write anywhere. The program validates form, not
  authorization.
- **Pubkey-gated** — writes are valid only if signed by a key the URI
  identifies. The program verifies the signature.
- **Managed** — a single operator (or quorum) approves writes. Useful for
  curated networks.
- **Content-conserved** — writes consume inputs and produce outputs; the
  program enforces conservation.

The model lives in the program. Different programs in the same schema can
use different models.

## Packaging a protocol

A protocol is shipped as a module that exports a schema (and any helpers
for app developers — typed wrappers, URI builders, etc.). Operators load
the schema into a node; app developers depend on the same module to get
typed access.

For the current packaging layout and the recommended module shape, follow
TARGETS.md to find the framework's current protocol-author docs on JSR.

## What to verify in code before writing it

When the user asks you to design a protocol, do not generate import
statements or function calls until you have done the relay in TARGETS.md.
The shape of programs and schemas is stable; the exact names and exports
are not.
