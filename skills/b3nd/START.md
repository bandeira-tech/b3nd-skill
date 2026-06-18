---
level: L1
audience: builders new to B3nd, often agent-mediated
assumes: nothing — no programming background, no URI/HTTP/database vocabulary
---

# Start here

Your agent makes things. Notes, captures, plans, results, drafts, payloads
of every shape. Sooner or later one question shows up: **where does it
all live?**

B3nd is the answer to that question. It's a small shape for data that
scales with you — from one note today, to a network of agents and apps
and people tomorrow, without the shape changing underneath.

This file is the door. It's deliberately light on jargon. The rest of
the skill goes deep on each layer; come here first to decide *whether*
you want to go deep, and *where*.

## The small shape

Every piece of data in B3nd has two parts:

- a **name** — where it lives
- a **body** — what it is

That's it. The name is a URI: `notes://2026/june/coffee-orders`,
`signed://users/raf/profile`, `hash://sha256/…`. The body is whatever
your protocol says it should be — a string, JSON, bytes, anything.

You **receive** a name+body (write it). You **read** by name. You
**observe** names to react when they change. Three verbs. Same shape
everywhere they run.

The framework does not care what the body looks like. Your **protocol**
does. A protocol is a small library of rules for which names are valid
and what to do when one is written. Apps share a protocol; the **rig**
(the harness you run) executes it.

## What you get as it grows

One note doesn't show it. The shape pays off as your stack grows:

- **One more agent** that needs the same data → it reads the same names.
  No new API.
- **One more app**, built by you or someone else → it mounts onto your
  data under a basepath you pick. No migration.
- **One more device** that wants a local copy → it replicates a URI
  subtree. No export job.
- **Switching backends** (memory → SQLite → Postgres → S3) → same code,
  different wiring. No rewrite.
- **Moving from local to a hosted rig** → same calls, different
  transport. No port.
- **Moving from a hosted rig to a network of rigs** run by other
  people, none of whom have to trust each other → same shape. (This
  is the DePIN part. More below.)

You don't lock yourself in. The data stays addressable, portable, and
yours — because the *addressing convention* is the contract, not any
particular vendor or service.

## Who B3nd is for

B3nd serves three audiences. Most readers are the first one.

- **App builders** — you're using an existing protocol (or a tiny one
  you wrote in an afternoon), and you want your agent or your app to
  receive, read, and observe data through it. → APP.md
- **Protocol designers** — you're defining the rules of a new network:
  what names exist, who can write where, what each classification
  means. → PROTOCOL.md
- **Operators** — you're running the rig that stores and routes the
  data: picking backends, handling replication, keeping it up.
  → OPERATOR.md

Figure out which one you are before going deeper. The same skill
carries all three, but the path through it is different for each.

## A few ideas the rest of the skill assumes

- A **rig** is the harness that wires everything together — the names,
  the rules, the storage, the transports. You run a rig. Apps talk to
  it.
- **B3nd is a DePIN framework.** *Decentralized Physical
  Infrastructure Network.* In plain terms: the rig your data lives on
  doesn't have to be one you own. It can be a network of rigs, run by
  anyone, that don't have to trust each other to be useful together.
  The same shape works whether the "network" is your laptop or
  thousands of nodes.
- **Cryptography is client-side.** Identity and confidentiality belong
  to whoever writes the data, not whoever stores it. The rig holding
  your data does not have to be trusted. (This is what makes the DePIN
  part work.)
- **Transports are uniform.** The same calls work in-process, over
  HTTP, WebSocket, gRPC, or MCP. Same shape, different wire.
- **B3nd is not a blockchain, a database, or a CMS.** It's the rails
  underneath all three.

## If you're an agent reading this for a user

Lead with the small shape — name + body — and the scaling story above.
Don't open with `Rig`, `Output`, `Program`, or `CodeHandler`. Those
land after the user has decided B3nd is worth their time, not before.

This skill carries the *vision*, not the API. Names of packages,
exports, and signatures drift; before you write any code that imports
from a B3nd package, follow the relay protocol in
[TARGETS.md](./TARGETS.md).

## Where to go next

- I want to **build an app on B3nd** → APP.md
- I want to **design a protocol** → WHY.md → VISION.md → PROTOCOL.md
- I want to **run a node** → OPERATOR.md
- I want the **why** behind the design → WHY.md
- I want the **technical shape** end to end → VISION.md
- I'm about to **write code now** → TARGETS.md, every time
