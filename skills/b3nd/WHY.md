# Why B3nd exists

In the old shape of software, every new capability ships as its own
service. UI → endpoint → library → database → back. Same work,
written again per feature. The user's data ends up scattered across
apps that don't see each other, exported (if at all) through APIs
the vendor controls.

B3nd inverts that. Computing is the commodity now; data is the asset.
You write the rules for the data — types, validators, transforms —
once, as pure functions. The rest (clients, servers, request
lifecycle, persistence, transport) is canned infrastructure wired
underneath them. Same rules across SQLite, Postgres, S3, IndexedDB.
Same calls across in-process, HTTP, WebSocket, gRPC, MCP. Different
deployment, same code.

The user gets their data back. Apps stop being silos for what they
generate; the rig the user runs is the substrate, and any app that
follows the addressing convention can compose on top.

## The production drag

Every service in production demands attention forever. Security
updates, deploy pipelines, on-call, schema migrations, export APIs
nobody trusts. For a team it's the slow tax that explains why
velocity decays. For a solo builder it's fatal — three small apps
generate enough operational drag to consume the time meant for the
fourth.

SaaS, more hires, more AI shift the burden without removing it.
AI-accelerated SOA deploys more surface, not less, and the operator
pays for all of it.

## What B3nd swaps in

Instead of one service per capability, write one **protocol** per
domain:

- **Programs** classify each incoming `[name, body]` and return a
  code.
- **Handlers** transform classified outputs into the writes to
  persist or refuse.
- The **rig** runs both, handles transport and storage, broadcasts
  changes to observers.

Three primitives on the rig — `receive`, `read`, `observe` — plus
`status` for discovery. Every system does the same four things with
data: move it, save it, send it onward, show it. The rig collapses
those to one surface that works the same everywhere it runs.

## The primitives came from DePIN

The path to this shape ran through DePIN, blockchains, and Bitcoin.
B3nd is not any of those (see VISION.md), but those communities had
done the hard thinking on the primitives that turn out to be right
for any data-owning system:

- **URI-keyed addressing.** A canonical name owned by no service.
- **Content addressing.** The hash is the locator; replication is
  trivial; integrity is self-evident.
- **Client-side cryptography.** Identity and confidentiality belong
  to the writer. Nodes are untrusted by design.
- **Composable protocols.** Rules are libraries, not services.

Apply them to a personal productivity tool and the user gets the
same data-ownership posture a public network gets.

## What this produces in the design

- **Programs and handlers are pure functions.** All the business
  logic; none of the wiring. (VISION.md, PROTOCOL.md)
- **The rig owns the wire.** App and protocol code never couple to
  a backend or a transport. (VISION.md, OPERATOR.md)
- **Behavior-named schemes.** `immutable://`, `signed://`,
  `encrypted://` describe rules, not domains. Apps compose by
  agreeing on the rule. (PROTOCOL.md)
- **URI-subtree records.** Decompose a record into a tree of URIs;
  current state is a fold over the listing. (PROTOCOL.md)
- **Mount basepaths so users keep control.** Protocols are
  factories. The user runs the rig; apps mount onto it. (PROTOCOL.md)
- **status() as a manifest.** Runtime self-description. (PROTOCOL.md,
  MCP.md, FRONTEND.md)
- **Same module on both sides.** Browser, server, and MCP run the
  same protocol module. (PROTOCOL.md, FRONTEND.md, MCP.md)

Each exists because the data — not the service — is the unit of
composition, and the user — not the app vendor — owns it.

## Where to go next

- The framework's technical shape → VISION.md
- Designing a protocol → PROTOCOL.md
- Building an app → APP.md, then FRONTEND.md or MCP.md
- Running nodes → OPERATOR.md
- Current API symbols → TARGETS.md
