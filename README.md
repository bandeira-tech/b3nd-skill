# b3nd-plugin

A Claude Code plugin that teaches the **shape** of [B3nd](https://github.com/bandeira-tech/b3nd)
and instructs the agent to verify everything else against live sources.

B3nd's APIs evolve faster than any embedded skill content can keep up
with. So this plugin deliberately ships:

- Stable, vision-level content about the framework (three roles, the
  message tuple, programs, schemas, trust models).
- A **relay protocol** the agent follows before writing any code:
  confirm the user's installed versions, fetch current exports from
  JSR or the source repos, then proceed.

It does not ship API signatures, import paths, or version-pinned code
examples. Those live in the source.

## Install

```
/plugin marketplace add bandeira-tech/b3nd-plugin
/plugin install b3nd
```

## What's inside

```
.claude-plugin/         — plugin + marketplace manifest
skills/b3nd/
  SKILL.md              — entry point + routing
  VISION.md             — what B3nd is and why
  PROTOCOL.md           — designing a protocol
  APP.md                — building an app on a protocol
  OPERATOR.md           — running a node
  TARGETS.md            — current packages, versions, relay protocol
```

## Updating

When B3nd's package list, naming, or major patterns change, update
`skills/b3nd/TARGETS.md` and bump the plugin version. Conceptual files
(`VISION.md`, `PROTOCOL.md`, `APP.md`, `OPERATOR.md`) should rarely
need to change — they describe the framework's shape, not its API.

## License

MIT.
