# Targets and relay protocol

This file is the bridge between the skill's vision content and the
moving target of B3nd's actual APIs. **Read this every time you are
about to write code that imports from a B3nd package.**

## Current ecosystem

B3nd is currently split across three load-bearing repos, with a few
auxiliary packages around them.

| Package                          | Repo                  | Role                                                               |
| -------------------------------- | --------------------- | ------------------------------------------------------------------ |
| `@bandeira-tech/b3nd-core`       | `b3nd-core`           | Framework foundation. Rig, PIN, identity, types, encoding, network. **Start here.** |
| `@bandeira-tech/b3nd-move`       | `b3nd-move`           | Transport. HTTP, WebSocket, gRPC-over-HTTP, MCP services + clients. |
| `@bandeira-tech/b3nd-save`       | `b3nd-save`           | Storage. Postgres, Mongo, SQLite, FS, IPFS, S3, Elasticsearch, LocalStorage, IndexedDB, memory + Store→PIN clients. |
| `@bandeira-tech/b3nd-cli`        | `b3nd-cli`            | The `bnd` CLI. Keygen, send, read, list, node ops.                 |
| `@bandeira-tech/b3nd-web-rig`    | `b3nd-web-rig`        | Browser dev rig + dashboard. Useful for app authors learning B3nd. |

All under `https://github.com/bandeira-tech/`.

## Target versions (as of this skill release)

These are the versions this skill's conceptual content was checked
against.

- `@bandeira-tech/b3nd-core` — 0.22.x
- `@bandeira-tech/b3nd-move` — 0.17.x
- `@bandeira-tech/b3nd-save` — 0.8.x
- `@bandeira-tech/b3nd-cli` — 0.2.x

Versions move. Use the relay protocol below before treating any of
these as current.

## Packages to avoid leaning on

- **`@bandeira-tech/b3nd-canon`** — very early. Exists but not the
  stable surface to build against yet. Canon is heading toward the
  patterns this skill is teaching: mounting basepaths, URI-subtree
  records, `status()` as a manifest, behavior-named foundational
  schemes (`immutable://`, `signed://`, `mutable://`, `encrypted://`).
  When you implement one of those patterns in a protocol you write
  here, treat the implementation as canon's draft — keep it small
  enough to lift in without a rewrite once canon stabilizes. If the
  user asks about canon now, flag that it's early and confirm
  before assuming any particular shape.
- **`@bandeira-tech/b3nd-sdk`** — currently dormant. The umbrella SDK
  package is not the recommended entry point right now. Use core,
  move, and save directly.
- **`@bandeira-tech/b3nd-servers`** — no longer exists. Its surface
  was absorbed into `b3nd-move`. If you encounter references to it in
  user code, treat them as needing migration.

## Relay protocol

Every time the user asks for code, follow these steps in order.

**1. Find the user's installed versions.**

Look at `deno.json`, `package.json`, or `import_map.json` for
`@bandeira-tech/*` entries. If none exist, ask the user whether they
have a B3nd project set up yet.

**2. Compare against the targets above.**

If the user's versions are within the target ranges, proceed. If
they're older or newer, tell the user — examples in the latest docs
may not match their pinned version, and you may need to fetch docs
for that specific version.

**3. Look up the current API for the package you're about to use.**

Pick a source, in this order of preference:

- **Local source** if the user has the repo cloned (typically
  `~/ws/b3nd-{core,move,save,cli,web-rig}/`). Read `deno.json`'s
  `exports` block and `mod.ts`/`src/.../mod.ts` for the actual
  symbols. Local source is the freshest truth.
- **JSR doc pages**: `https://jsr.io/@bandeira-tech/<pkg>` and
  `/doc`. JSR shows exports and types for the published version.
- **GitHub source**: `https://github.com/bandeira-tech/<repo>`. Look
  at `mod.ts`, `deno.json`, and `README.md`.
- **`deno info`** if the package is installed:
  `deno info jsr:@bandeira-tech/<pkg>@<version>`.

**4. Only now write code.**

Use the symbols and import paths you just verified. Do not fall back
to patterns you remember.

## When the user has the repos cloned

Common layout:

```
~/ws/b3nd-core/   — framework foundation
~/ws/b3nd-move/   — transport
~/ws/b3nd-save/   — storage
~/ws/b3nd-cli/    — the bnd CLI
```

If a project's `deno.json` has `imports` mapping `@bandeira-tech/b3nd-core`
(or move/save) to a relative path like `../b3nd-core/mod.ts`, the user
is in a workspace checkout — read those files directly.

## Things that have moved recently

Background to keep you from generating stale code. If you have a
memory of any of these in the old form, distrust it and relay.

- The **tuple is `[uri, payload]`** — two items. There is no
  separate "values" slot. Protocols derive whatever they need from
  the payload inside their own programs and handlers.
- The protocol model is **programs (classifiers returning codes) +
  code handlers (pure transforms returning `Output[]`)**. Both pure.
  There is no single "validation function returns accept/reject" any
  more.
- **`b3nd-servers` no longer exists** — transport lives in
  `b3nd-move`, including the MCP service.
- The umbrella **`b3nd-sdk` is dormant** — depend on `b3nd-core`,
  `b3nd-move`, `b3nd-save` directly.
- The core interface is **`ProtocolInterfaceNode` (PIN)**, with four
  primitives: `receive`, `read`, `observe`, `status`.
- Reads and observes take **locators** (opaque, executing-client's
  grammar), not URIs. Writes take URIs.
- **`b3nd-canon` is early** — do not anchor code in it without
  confirming with the user.

When in doubt, relay.

## When relay fails

If JSR is unreachable, the GitHub repo is private, and the user does
not have the source locally, say so plainly. Do not invent function
names. Offer to proceed against the closest version of the docs you
can fetch and flag the assumption to the user.
