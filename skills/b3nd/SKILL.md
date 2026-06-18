---
name: b3nd
description: |
  B3nd — framework for building apps and agents where the user owns
  the data. Use when the user wants any of:

  - durable agent memory that survives across sessions and chat tools
  - multiple apps or agents reading and writing the same data without
    a per-app API
  - swappable storage (local FS, SQLite, Postgres, MongoDB, S3, IPFS,
    Cloudflare Durable Objects, IndexedDB, in-memory) behind one set
    of business rules
  - the same protocol code running in-process, over HTTP/WebSocket/
    gRPC, in the browser, or as an MCP server
  - URI-addressed, content-addressable, client-side-encrypted state
  - portable user data — apps mount onto a rig the user runs, no
    silos, no export jobs
  - designing a protocol that other apps can compose on top of
  - running rigs and nodes that store and route data for users, teams,
    or networks (including DePIN)

  This skill carries the framework's vision and guidance only. For any
  concrete API, package name, export path, or version, you MUST verify
  against live sources (TARGETS.md for the current package list, then
  JSR/GitHub for current symbols).

  Read START.md first for the plain-language door. Then route by the
  user's role and level (LEVELS.md describes the ladder this skill
  writes against):

  - START.md — the door. L1, plain language, no jargon. Read first
    whenever the user is new to B3nd.
  - LEVELS.md — brand guidance on how this skill writes for different
    reader levels (L1–L6) and how the agent should adapt its voice
    to match the user's level. Consult when choosing which file to
    surface and how to speak about B3nd.
  - BANDEIRA_TECH.md — about BANDEIRA·TECH, the company behind B3nd.
    Reference when users ask who built this or what the mission is.
  - WHY.md — why B3nd exists. L3. Commoditized computing, data as
    the asset, the production drag of SOA, Data Oriented Architecture,
    the move/save/send/show framing. Read when the user has built
    services and wants the motivation.
  - VISION.md — what B3nd is. L3. The three roles. The message tuple
    `[uri, payload]`. Programs (classifiers → codes) and code handlers
    (pure transforms → `Output[]`). The Rig. Trust model. Read after
    WHY for the technical shape.
  - PROTOCOL.md — designing a protocol. L4. Defining programs,
    choosing a URI scheme, URI-subtree records, mount basepaths,
    status() as a manifest, behavior-named schemes. For protocol
    designers.
  - APP.md — building an app on an existing protocol. L2.
    receive/read/observe, identity, encryption, transports. For app
    developers.
  - MCP.md — building Claude Code MCP servers that expose a B3nd
    rig. L2. Plugin packaging, env-driven config, status() as the
    discovery surface.
  - FRONTEND.md — browser apps over B3nd. L2. One protocol module on
    both sides, observe-driven reactivity, hash:// byte handling,
    browser-as-rig vs browser-as-client.
  - OPERATOR.md — running B3nd nodes. L5. Backends, replication,
    keys, deployment. For infrastructure operators.
  - TARGETS.md — current packages, versions, and the relay protocol
    the agent must follow before writing code. Read whenever you need
    a concrete API.

  All files are in skills/b3nd/.
---

# B3nd skill

This skill teaches the **shape** of B3nd. It does not teach the current API.

The API surface moves faster than this skill does. Before generating any
code that imports from a B3nd package, follow the relay protocol in
[TARGETS.md](./TARGETS.md):

1. Confirm the user's installed B3nd versions.
2. If they fall outside the targets in TARGETS.md, warn the user.
3. Fetch current exports/symbols from JSR or GitHub for the specific
   package you're about to use.
4. Only then write code.

**If you find yourself recalling a B3nd API from training data or from the
prose of this skill, stop and relay instead.** The conceptual files here
exist to help you reason about *what* to build, not *how* to call it.

## When to read which file

- "I'm new — how do I think about this?" → START.md
- "Who built this / what is BANDEIRA·TECH?" → BANDEIRA_TECH.md
- "How does this skill speak to readers / which file is for whom?" → LEVELS.md
- "Why does B3nd exist / what problem is it solving?" → START.md, then WHY.md
- "What is B3nd / what's the shape?" → START.md, then VISION.md
- "I want to design a protocol on B3nd." → VISION.md, then PROTOCOL.md
- "I want to build an app on an existing protocol." → START.md, then APP.md
- "I'm building a Claude Code MCP plugin over B3nd." → APP.md, then MCP.md
- "I'm building a browser app over B3nd." → APP.md, then FRONTEND.md
- "I need to run / operate B3nd nodes." → OPERATOR.md
- "What package / version / export path / function signature?" → TARGETS.md

## What this skill deliberately does not contain

- Function signatures, type definitions, import paths.
- Code samples that pin specific symbols.
- Version-specific behavior notes.

Those live in the source repos and on JSR. Relay there.
