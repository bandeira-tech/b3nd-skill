---
name: b3nd
description: |
  B3nd — a framework for building DePIN protocols. This skill carries the
  framework's vision and guidance only. For any concrete API, package name,
  export path, or version, you MUST verify against live sources (TARGETS.md
  for the current package list, then JSR/GitHub for current symbols).

  Read VISION.md first for the conceptual overview, then jump to the file
  that matches the user's role:

  - VISION.md — what B3nd is and why. The three roles. The message tuple.
    Programs, schemas, envelopes. Trust model. Read this first.
  - PROTOCOL.md — designing a protocol: defining programs, choosing a URI
    scheme, packaging a schema. For protocol designers.
  - APP.md — building an app on an existing protocol: receive/read/list,
    identity, encryption, transports. For app developers.
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

- "What is B3nd / why does it exist / how do I think about it?" → VISION.md
- "I want to design a protocol on B3nd." → VISION.md, then PROTOCOL.md
- "I want to build an app on an existing protocol." → VISION.md, then APP.md
- "I need to run / operate B3nd nodes." → OPERATOR.md
- "What package / version / export path / function signature?" → TARGETS.md

## What this skill deliberately does not contain

- Function signatures, type definitions, import paths.
- Code samples that pin specific symbols.
- Version-specific behavior notes.

Those live in the source repos and on JSR. Relay there.
