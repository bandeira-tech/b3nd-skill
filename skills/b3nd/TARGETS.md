# Targets and relay protocol

This file is the bridge between the skill's stable vision content and the
moving target of B3nd's actual APIs. **Read this every time you are about
to write code that imports from a B3nd package.**

## Current packages

| Package                       | Registry | Role                                                                 |
| ----------------------------- | -------- | -------------------------------------------------------------------- |
| `@bandeira-tech/b3nd-sdk`     | JSR      | Umbrella SDK for Deno/servers. Re-exports core + canon.              |
| `@bandeira-tech/b3nd-web`     | NPM      | Browser umbrella. Same surface as the SDK, bundled for browsers.     |
| `@bandeira-tech/b3nd-core`    | JSR      | Framework foundation: types, encoding, rig, identity, network.       |
| `@bandeira-tech/b3nd-canon`   | JSR      | Protocol toolkit: message envelopes, hash, auth, encryption.         |
| `@bandeira-tech/b3nd-cli`     | JSR      | `bnd` CLI for keygen, send, read, list, node ops.                    |
| `@bandeira-tech/b3nd-servers` | JSR+NPM  | Server composition + transports (HTTP, gRPC). Holds the MCP server.  |

Repos live under `https://github.com/bandeira-tech/` with matching names.

## Target versions (as of this skill release)

These are the versions the conceptual content in this skill was checked
against. If the user's project pins something materially older, warn them
that examples in the live docs may not match.

- `@bandeira-tech/b3nd-sdk` — 0.9.x
- `@bandeira-tech/b3nd-web` — 0.9.x
- `@bandeira-tech/b3nd-core` — 0.22.x
- `@bandeira-tech/b3nd-canon` — 0.13.x
- `@bandeira-tech/b3nd-cli` — 0.2.x

These move. Use the relay protocol below before treating them as current.

## Relay protocol

Every time the user asks for code, follow these steps in order.

**1. Find the user's installed versions.**

Look at their `deno.json`, `package.json`, or `import_map.json` for
`@bandeira-tech/*` entries. If none exist, ask the user whether they have
a B3nd project set up yet.

**2. Compare against the targets above.**

If the user's versions are within the target ranges, proceed. If they're
older, tell the user — examples you might find in the latest docs may not
work on their version, and you may need to fetch docs for their specific
version.

**3. Look up the current API for the package you're about to use.**

Pick one source, in this order of preference:

- **JSR doc pages**: `https://jsr.io/@bandeira-tech/<pkg>` (and
  `/doc` for symbols). JSR shows the exports and types for the published
  version. Use `WebFetch`.
- **GitHub source**: `https://github.com/bandeira-tech/<repo>` — look at
  `mod.ts`, `deno.json` (`exports` block), and any `README.md`. Use
  `WebFetch`.
- **`deno info`**: if the user has the package installed, run
  `deno info jsr:@bandeira-tech/<pkg>@<version>` via Bash to list its
  module graph.

**4. Only now write code.**

Use the symbols and import paths you just verified. Do not fall back to
patterns you remember — names have changed across the recent
core/canon/cli/web-rig/servers extractions.

## When the user has the repos cloned

If the user works in `~/ws/b3nd*` (or any sibling layout with the source
repos checked out), prefer reading their local `deno.json` and `mod.ts`
over JSR. Local source is always the freshest truth.

Common layout:

```
~/ws/b3nd/        — SDK umbrella (this plugin's parent project)
~/ws/b3nd-core/   — core foundation
~/ws/b3nd-canon/  — protocol toolkit
~/ws/b3nd-cli/    — bnd CLI
```

If the SDK's `deno.json` has `imports` mapping `@bandeira-tech/b3nd-core`
to a relative path like `../b3nd-core/mod.ts`, the user is in a workspace
checkout — read those files directly.

## Things that have moved recently

Background to keep you from generating stale code. If you have a memory
of any of these in the old form, distrust it and relay.

- **MCP server** moved into `@bandeira-tech/b3nd-servers`, not the SDK.
- **CLI** is now `@bandeira-tech/b3nd-cli` and is wrapped by `./bnd` in
  the SDK repo.
- **Web rig** moved out of the SDK to its own repo.
- The core interface is now named `ProtocolInterfaceNode` (PIN), not
  `NodeProtocolInterface`.
- The message tuple is `[uri, values, data]` (3-tuple), not `[uri, data]`.
- `DataClient` was renamed to `MessageDataClient`.

When in doubt, relay.

## When relay fails

If JSR is unreachable, the GitHub repo is private, and the user does not
have the source locally, say so plainly. Do not invent function names.
Offer to proceed against the closest version of the docs you can fetch
and flag the assumption to the user.
