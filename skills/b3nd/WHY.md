# Why B3nd exists

Software development has entered an era where computing is commodity
and data is the asset. The chips that run your code, the storage that
holds it, the network that ships it — all of it is a button on a
console. What is still scarce is the data, the rules that shape it,
and the relationships the data carries.

That shift hits the stack at two heights.

At the **architecture and operations** layer, data is the top thing.
The question stops being "what service do we deploy" and starts being
"where does the data live, who can read it, who can derive from it,
how does it move." Backends become interchangeable; addressing,
authorization, replication, and content provenance become the load-
bearing concerns.

At the **application development** layer, the work is to commoditize
data workflows themselves. A new business capability should not need
a new service surface. It should be the rules that govern its data —
written once, swappable across backends, addressable by the user, and
extensible by anyone else willing to follow the same conventions.
That is what lets users own the data their apps generate.

## The production drag

For most of the last decade every new capability shipped as its own
service. UI client → HTTP request → library → database format → back.
Each layer is mostly the same kind of work — managing data — written
again for one more domain. The ratio of business rules to boilerplate
is tiny, and the operational drag is permanent: every service in
production demands attention, money, security updates, and cognitive
room forever.

SaaS, more hires, more AI shift the burden around without removing
it. Service-oriented architecture, accelerated by AI's writing speed,
deploys more surface than ever — and the operator pays for all of it.

For a solo builder running several products that drag is fatal. For
a team it is the slow tax that explains why velocity decays.

## Data Oriented Architecture

The alternative is to invert the unit of composition. Instead of a
service per capability, write the **data rules** for the capability —
types, validators, transforms — as pure functions. Then mix and match
canned boilerplate for everything else: clients, servers, request
lifecycle, persistence, sinks. The same business rules pair with
local files, a Durable Object, Postgres, or Mongo without rewrites.
The same rules run in the browser, on a server, behind an MCP tool —
the surface follows the data, not the other way around.

B3nd is the framework that makes that work. A protocol is the rules
(programs that classify and handlers that transform). A rig is the
harness that wires the rules to backends and transports. Apps and
operators share one protocol module; the rig adapts to wherever it
runs.

## Move, save, send, show

Every system does the same four things with data: move it between
places, save it durably, send it onward to interested parties, and
show it to humans or other programs. B3nd collapses that to four
primitives on the rig — `receive`, `read`, `observe`, `status` — and
ships everything else (transports in `b3nd-move`, backends in
`b3nd-save`, the harness in `b3nd-core`) as pluggable infrastructure
around them.

That is the surface every app gets. The same four calls work
in-process, over HTTP, WebSocket, gRPC, or MCP. The same four calls
serve a browser tab, a CLI, a server, or an agent.

## Why DePIN primitives

The path to this framing ran through DePIN, blockchains, and Bitcoin.
Not because B3nd is a blockchain — it is not, see VISION.md — but
because those communities had already done the hard thinking on the
primitives this problem needs:

- **URI-keyed addressing.** A canonical name for every piece of
  state, owned by no service.
- **Content addressing.** The hash *is* the locator; data integrity
  is self-evident, replication is trivial.
- **Client-side cryptography.** Identity and confidentiality belong
  to the writer, not the host. Nodes are untrusted by design.
- **Composable protocols.** Rules are libraries, not services.

Apply those primitives to a single-user productivity tool and the
same shape that secures a public network gives you a personal data
plane the user fully owns.

## What this produces in the design

The rest of this skill is the consequences:

- **Programs and handlers are pure functions.** All the business
  logic; none of the wiring. (VISION.md, PROTOCOL.md)
- **The rig owns the wire.** App and protocol code never couple to a
  backend or a transport. (VISION.md, OPERATOR.md)
- **Behavior-named schemes.** `immutable://`, `signed://`,
  `encrypted://` describe rules, not domains. Apps compose by
  agreeing on the rule. (PROTOCOL.md)
- **URI-subtree records.** Decompose a record into a tree of URIs;
  current state is a fold over the listing. No read-modify-write,
  no schema mismatches on partial updates. (PROTOCOL.md)
- **Mount basepaths so users keep control.** Protocols are factories
  parameterised by where on the user's rig they live. The user runs
  the rig; apps mount onto it. (PROTOCOL.md, VISION.md "Users own
  the data")
- **status() as a manifest.** The rig publishes its own schema,
  vocabulary, examples. Skills, MCP tools, UIs read from one place.
  (PROTOCOL.md, MCP.md, FRONTEND.md)
- **Same module on both sides.** Browser, server, and MCP all run
  the same protocol module. Refusals surface without a round trip.
  (PROTOCOL.md, FRONTEND.md, MCP.md)

Each of those exists because the data — not the service — is the
unit of composition, and the user — not the app vendor — owns it.

## Where to go next

- The shape of B3nd as a framework → VISION.md
- Designing a protocol → PROTOCOL.md
- Building an app → APP.md, then FRONTEND.md or MCP.md
- Running nodes → OPERATOR.md
- Current API symbols → TARGETS.md
