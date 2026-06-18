---
name: b3nd
description: |
  B3nd — a framework for building DePIN protocols. This skill carries the
  framework's vision and guidance only. For any concrete API, package name,
  export path, or version, you MUST verify against live sources (TARGETS.md
  for the current package list, then JSR/GitHub for current symbols).

  Read WHY.md first for the motivation, then VISION.md for the shape,
  then jump to the file that matches the user's role:

  - WHY.md — why B3nd exists. Commoditized computing, data as the
    asset, the production drag of SOA, Data Oriented Architecture,
    the move/save/send/show framing, and why DePIN primitives turned
    out to be the right answer. Read first.
  - VISION.md — what B3nd is. The three roles. The message tuple
    `[uri, payload]`. Programs (classifiers → codes) and code handlers
    (pure transforms → `Output[]`). The Rig. Trust model. Read second.
  - PROTOCOL.md — designing a protocol: defining programs, choosing a URI
    scheme, packaging a schema. For protocol designers.
  - APP.md — building an app on an existing protocol: receive/read/list,
    identity, encryption, transports. For app developers.
  - MCP.md — specifics for building Claude Code MCP servers that expose
    a B3nd rig. Plugin packaging, env-driven config, status() as the
    discovery surface.
  - FRONTEND.md — specifics for building browser apps over B3nd. One
    protocol module on both sides, observe-driven reactivity, hash://
    byte handling, browser-as-rig vs browser-as-client.
  - OPERATOR.md — running B3nd nodes: backends, replication, keys,
    deployment. For infrastructure operators.
  - TARGETS.md — current packages, versions, and the relay protocol the
    agent must follow before writing code. Read this whenever you need a
    concrete API.

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

- "Why does B3nd exist / what problem is it solving?" → WHY.md
- "What is B3nd / how do I think about it?" → WHY.md, then VISION.md
- "I want to design a protocol on B3nd." → WHY.md, VISION.md, then PROTOCOL.md
- "I want to build an app on an existing protocol." → VISION.md, then APP.md
- "I'm building a Claude Code MCP plugin over B3nd." → APP.md, then MCP.md
- "I'm building a browser app over B3nd." → APP.md, then FRONTEND.md
- "I need to run / operate B3nd nodes." → OPERATOR.md
- "What package / version / export path / function signature?" → TARGETS.md

## What this skill deliberately does not contain

- Function signatures, type definitions, import paths.
- Code samples that pin specific symbols.
- Version-specific behavior notes.

Those live in the source repos and on JSR. Relay there.
