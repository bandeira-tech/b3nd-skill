---
level: L2
audience: developers building Claude Code MCP plugins over B3nd
assumes: familiarity with Claude Code plugins (commands/skills/.mcp.json), stdio JSON-RPC; APP.md absorbed
---

# Building MCP servers over B3nd

A B3nd rig becomes a Claude Code MCP server by exposing it over stdio
through `b3nd-move`'s MCP service. The MCP surface stays uniform with
every other B3nd transport — three tools, the URI/payload shape, the
rig is the contract.

This file teaches the patterns. For the current API surface (package
names, export paths, helper signatures), follow the relay protocol in
TARGETS.md.

## Surface

The MCP server publishes three tools backed directly by the rig:

- `b3nd_receive` — send writes as `[uri, payload]` tuples.
- `b3nd_read` — read URIs (point reads or `?fn=...` locators).
- `b3nd_status` — return the rig's status manifest.

Bespoke `create_task` / `list_tasks` tools are not the pattern. The
protocol's URI grammar **is** the API; the skill teaches the grammar.
Higher-order verbs (e.g. a single `task://t/create` write that
decomposes into multiple stored URIs) belong inside protocol programs
and handlers, not in the MCP surface.

## Boot script shape

A typical MCP entry script:

1. Read env for basepath and storage root.
2. Construct a Rig with the protocol's programs/handlers wired to the
   chosen mount.
3. Hand the rig to the MCP service and connect it to stdio.

Status chatter goes to stderr; stdout is reserved for MCP JSON-RPC.
A single print to stdout outside the protocol corrupts the channel.

## Env-driven configuration

Both basepaths — URI mount and storage root — should be env vars with
sane defaults. Two patterns worth holding to:

- Check `env && env.length > 0` before falling back to a default. The
  `??` operator only catches null/undefined; an empty string from a
  `${VAR:-}` substitution in `.mcp.json` flows through and bypasses
  the default.
- Surface the resolved basepath in `b3nd_status` output so the agent
  can confirm at runtime what mount it's talking to.

When the user wants to remount or relocate storage, the agent's path
is: read `.mcp.json`, edit the `env` block, tell the user to restart
the MCP. Live remount is a future capability.

## status() as the discovery surface

The agent uses `b3nd_status` to learn the protocol at runtime.
Hydrate the status payload with:

- The protocol's schema(s).
- The protocol's code vocabulary (`ok:*`, `refuse:*`).
- A handful of URI examples (point reads, listing locators).
- Mount metadata (basepath, store kind, storage root).

The skill text then becomes a thin overlay on top of what status
already tells the agent — and downstream consumers (UI catalog,
agent prompts, dashboards) read from one place.

## Same protocol module on both sides

If the MCP fronts a remote rig (`*_BACKEND`-style env pointing at an
HTTP node), keep the protocol module itself in the MCP process. The
program classifies locally before delegating writes to the remote.
Refusals surface without a round trip; the wire only carries
accepted output.

## Plugin packaging pitfalls worth knowing

- **Plugin cache keys on declared version.** Bump the version on any
  layout change or `/plugin update` will report "already at latest"
  and skip the refresh.
- **Tool prefix includes plugin name AND MCP server name.** Tools
  surface as `mcp__plugin_<plugin>_<server>__b3nd_*`. Pick a server
  name distinct from the plugin name so the prefix is not
  `mcp__plugin_foo_foo__b3nd_read`.
- **`commands/` and `skills/` live at the repo root**, alongside
  `.claude-plugin/`, not under it.

These are Claude Code packaging concerns, not B3nd ones, but they
bite the same agent that's writing the rig.

## Where to look next

- Designing the protocol — PROTOCOL.md
- App-side patterns shared with frontend apps — APP.md
- Running the rig backend (storage, replication, identity) — OPERATOR.md
- Current API symbols — TARGETS.md
