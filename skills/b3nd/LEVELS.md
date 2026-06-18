# Levels

Brand guidance for writing learning files in this skill.

## Why levels exist

For a long time docs have tried to speak to everyone at once. The
price is a wall of text where the reader filters out 80% of the
material to reach the 20% they needed. Worse, it forces a single
voice — usually the senior-developer voice — onto material other
readers can't act on without a translator.

This skill commits to the opposite: **every file declares one level
and writes for that level only.** The agent reading this skill uses
the declared level to pick the right file for the user's context,
and adapts its own voice to match.

Don't mix levels in one file. Don't water down a higher level to
make it accessible. Don't bolt jargon onto a lower level to sound
rigorous. If a topic genuinely spans levels, write it twice.

## The shape of the ladder

L1–L3 are **progressive**. An L3 reader has been L2 and L1; they
don't need that material repeated, but the vocabulary and mental
models from earlier levels are fair game.

L4 and L5 are **specializations**. They sit beside the ladder, not
above it. A protocol designer (L4) is often also writing apps (L2);
a node operator (L5) almost always is. What makes them
specializations is the *role the reader is playing while reading*,
not their seniority.

L6 is the **philosopher's level**. Optional. Where the brand commits
to its ambitious claims, foreseen consequences, and intractable
tensions. Most files do not need L6 content. When L6 content is
required, it gets its own file (or a clearly labeled section at the
end of one).

## The levels

### L1 — Never built anything, agent-mediated

**Who.** Someone using an agent to build something for the first
time. May not know what an API is. May not have written code
outside of an agent conversation. Is *making* — notes, captures,
plans, a small app — but does not yet think of themselves as a
builder.

**Brings.** Curiosity, the agent, the question. A working laptop and
a chat window.

**Does not bring.** Vocabulary. Knowledge of HTTP, databases,
deployment, version control, dependencies, or protocols of any kind.

**Pain.** "Where did the thing go." "Can my next chat see it." "Is
it lost if I close this." "Why does each tool start me over."

**What B3nd offers them.** Their stuff has a name. Their stuff lives
somewhere they own — or on a network nobody owns, where no single
company can take it away from them. The next agent — and the agent
after that — can find it.

**Voice.** Direct. Concrete. One idea per sentence. Examples before
terms. No metaphors that require a prior craft.

**Vocabulary scope.** Plain English. *Place, name, thing, save, ask
for it back.* Never `URI`, `payload`, `rig`, `program`, `handler`,
`transport`, `backend` cold. If you must use one, define it inline
in half a sentence.

**What good looks like.**

> Your agent made a note. The note has a name — like a label on a
> folder — that says where it lives. When you (or another agent) ask
> for that name later, the note comes back. That's the whole idea.

### L2 — Has built apps with popular architecture

**Who.** Has shipped at least one CRUD app. Knows Express or Rails
or Next or Django or similar. Has used Postgres or MongoDB or
SQLite. Thinks naturally in endpoint → service → database. Has
probably integrated a third-party API.

**Brings.** A working stack-mental-model. Comfort with code. The
ability to read a function signature.

**Does not bring.** Operational experience at scale. Familiarity
with content addressing, cryptographic identity, or DePIN
primitives. May never have heard "data-oriented" used as a phrase.

**Pain.** Every new tool wants its own data silo. The user's data
is scattered across SaaS exports. Integrations are
write-once-then-rot. Each app they make is a service surface
forever.

**What B3nd offers them.** A data plane underneath the apps. One
rig, many apps, same data. The Express handler they would have
written becomes a tiny pure function in a protocol; the rig handles
the wire. The rig can be local (a SQLite file), hosted (their VPS),
or shared — a network of rigs run by other people, none of whom
have to trust each other. (That's the *DePIN* part of B3nd:
**Decentralized Physical Infrastructure Network.** Same code wires
to any of these.)

**Voice.** Comparative. Show the familiar shape first, then show
the B3nd shape next to it. Acknowledge the existing instinct ("this
is where you'd reach for an Express handler"). Honor the cost
they're trying to escape.

**Vocabulary scope.** `URI`, `rig`, `protocol`, `program`,
`handler`, `receive`/`read`/`observe`. Land cold once defined.
`Output`, `CodeHandler`, `ProgramResult`, `connection`, `route` —
define on first use.

**What good looks like.**

> In a typical Express app, a `POST /notes` handler validates the
> body, writes to a database, and returns. In B3nd, the validation
> is a **program** (a pure function that classifies the incoming
> `[name, body]`). The write is a **handler** (a pure function that
> returns what to persist). The rig owns the request lifecycle, the
> storage, the response — everything you used to write per-route.

### L3 — Architecture and SOA operations

**Who.** Has run multiple apps in production. Has paid the
operational tax — alerting, deploy pipelines, on-call. Has felt the
drag of services-per-capability. Probably owns or strongly
influences architectural decisions.

**Brings.** Vocabulary for systems work. An honest accounting of
what SOA costs. Strong opinions, weakly held, about CQRS, event
sourcing, replication, idempotency, schema evolution.

**Does not bring.** Familiarity with DePIN or content addressing as
a primary tool. May have written off blockchain-adjacent work as
inapplicable to product engineering. The brand voice at L3 names
that prejudice and pushes through it: DePIN primitives — URI-keyed
addressing, content addressing, client-side crypto, untrusted
hosts — are the load-bearing answer to the SOA drag this reader is
feeling. Don't soft-pedal them; they are the point.

**Pain.** Velocity decay. The slow tax of every service in
production. The dawning sense that AI-accelerated SOA deploys more
surface, not less. Nowhere obvious to spend energy that reduces the
burden.

**What B3nd offers them.** Data Oriented Architecture as the actual
alternative — not as a slogan but as a substrate. Programs and
handlers as the unit of business logic; the rig as the only
operational unit. The same module runs in browser, server, and
MCP — refusals surface without a round trip.

**Voice.** Honest about the drag. Names the tax. Uses words like
*decay, drag, load-bearing, moat.* Treats the reader as a peer who
has already done the hard thinking on tradeoffs.

**Vocabulary scope.** All of L2, plus `routes`, `hooks`, `events`,
`reactions`, `PIN`, `ProtocolInterfaceNode`. Free use of
architecture terms (CQRS, SOA, CRDT, replication topology) without
translation.

**What good looks like.**

> Service-per-capability multiplies the operational surface. Every
> endpoint is forever; every database migration is a coordination
> problem; every export API is a small political negotiation with
> the user. The rig collapses all of that to one harness with
> pluggable parts. A protocol is a library, not a service. There is
> no service surface to keep alive.

### L4 — Protocol designers

**Who.** Defining the rules of a network. Thinking about which
names exist, who can write where, what each classification code
means, how state is derived from a URI subtree.

**Brings.** Precision. Care for vocabulary. Willingness to think in
terms of pure functions and dispatch tables.

**Does not bring.** Necessarily — operational experience or prior
B3nd familiarity. But by the time someone is at L4, the *shape* of
B3nd has to fit in their head.

**Pain.** Designing a protocol that won't paint itself into a
corner. Choosing a URI scheme that names a *behavior* rather than a
*domain*. Avoiding the temptation to model state as a single
document when a URI subtree would be cleaner.

**What B3nd offers them.** A small, sharp abstraction with strict
semantics. Behavior-named schemes (`immutable://`, `signed://`,
`encrypted://`). URI subtree records. `status()` as a manifest. The
same protocol module runs everywhere the rig runs.

**Voice.** Precise. Vocabulary-disciplined. Names tradeoffs
explicitly. Distinguishes URIs from locators, programs from
handlers, names from bodies. Comfortable with formal language;
allergic to hand-waving.

**Vocabulary scope.** Everything. This is the level where the
technical vocabulary is *load-bearing*, not decorative.

**What good looks like.**

> A `signed://` scheme commits the rig to enforcing payload
> signatures matching the principal embedded in the URI. The
> program that owns the scheme verifies the signature, returns
> `SIGNED_OK` or `SIGNED_REJECT`, and is forbidden — by convention
> — from inspecting the payload's semantic content. That separation
> is what lets a `signed://` URI compose with any payload shape.

### L5 — Node operators

**Who.** Running rigs in production (or planning to). Picking
backends. Wiring replication. Handling keys. Keeping it up.

**Brings.** Strong opinions about storage, durability, replication
topology, ops surface, cost. Probably an existing platform
(Cloudflare, AWS, a homelab) they want B3nd to live on.

**Does not bring.** Necessarily — protocol design experience. Cares
about the *contract* the rig exposes more than the rules running
inside it.

**Pain.** Knowing what part of the system to blame when something
is slow or wrong. Knowing what's safe to swap and what's
load-bearing. Understanding what gets backed up and what doesn't.

**What B3nd offers them.** A clean contract: the rig is the only
operational unit. Backends are pluggable behind it. Nodes are
untrusted by design — they don't have to be hardened against
malice, only against loss. The same rig serves public and private
data without configuration. This is the operational shape of a
DePIN: untrusted hosts, content-addressed data, client-side keys.
The operator's job is uptime and durability, not gatekeeping.

**Voice.** Contractual. Specific about what the rig guarantees and
what it does not. Names which decisions are reversible and which
are not. Tables, not paragraphs, when there are choices to weigh.

**Vocabulary scope.** All technical vocabulary, plus operations
vocabulary (RPO/RTO, durability classes, fan-out, hot/cold storage,
key escrow).

**What good looks like.**

> The rig owns the wire; it does not own durability. A `b3nd-save`
> backend may be a single SQLite file (durable to disk, vulnerable
> to host loss), an S3 bucket (durable across availability zones,
> latency-dominated), or memory (not durable at all, useful for
> caches). Backends are swappable behind one `EntityStore` contract.
> Picking one is an operator decision, not an app decision.

### L6 — Evangelist / philosopher

**Who.** The brand itself, writing about itself. Or a reader who
already gets it and wants the long thought through to the end.

**Brings.** Patience for ambition. Tolerance for claims that can't
be proven by the morning's standup.

**Does not bring.** Necessarily — anything. L6 reaches forward,
not down.

**What L6 is for.**

- **Foreseen consequences.** *No moats on data.* *User as
  substrate.* *The political shape of computing-is-commodity-and-
  data-is-the-asset.* *DePIN as the inevitable form of consumer
  software once the addressing convention is the contract.*
- **Intractable tensions.** What stays unresolved on purpose, where
  the design refuses to choose, what the framework's restraint
  costs.
- **The longer arc.** Where this lands if it works.

**Voice.** Bolder. Denser. Allowed to make claims that aren't yet
operationally validated, as long as it's clear they are claims
about the *direction* and not the *current behavior*. Allowed to be
a little cracked.

**Vocabulary scope.** Free. Including coinages, including aesthetic
terms, including phrases not yet defined elsewhere — as long as
they're clearly L6 material and don't leak into L1–L5 files.

**What good looks like.**

> The default in most software is the opposite: each app owns its
> backend, each backend owns its database, and the user's data is
> whatever they can export through an API the app vendor controls.
> B3nd inverts that, and the inversion is permanent. Once the
> addressing convention is the contract, the vendor is downstream
> of the user. The political shape of the network follows.

## How to declare a level

Every learning file in this skill should declare its level near
the top. Either in YAML frontmatter or in a one-line note under
the title:

> **Level:** L2 — app builders. Assumes a working CRUD-stack
> mental model.

A file should not reach below or above its declared level. If a
paragraph from a higher level shows up in an L2 file, move it (or
its equivalent) to the higher-level file.

## How to mix levels across the skill

The reader navigates by level, not by file name. A graceful skill
has at least one file per level it claims to serve. Today this
skill skews L3–L5. `START.md` is the first L1-shaped surface;
adding a clean L2 file and pruning level-leakage from `VISION.md`,
`WHY.md`, and the role files is the obvious next step.

When in doubt, prefer:

- **More files, each tighter to its level**, over one file that
  tries to escalate gracefully.
- **A pointer between files** ("if this is too dense, see X" / "if
  this is too soft, see Y") over inline disclaimers.
- **Concrete examples written at the file's level**, over abstract
  definitions that could land at any level.

## Feedback to the writer (and the agent)

When reviewing a file against this guide, name the level mismatch
concretely:

- *"Paragraph 3 lands at L4 but the file is declared L2."* → Move
  or rewrite.
- *"This file is declared L1 but the first example uses `URI`
  cold."* → Translate or postpone.
- *"This whole file reads as L6 in L3 clothing."* → Split. Either
  commit to the L6 voice or strip the claims.

The brand goal is not to flatten everything to the simplest level.
It is to make the *level a choice* — a deliberate one — and let the
writing be as precise as that choice allows.
