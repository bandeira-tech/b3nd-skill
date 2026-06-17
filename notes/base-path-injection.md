# Base-path injection — having both cakes

*Follow-up to `uri-scheme-shape.md`. The behavior-named-scheme critique made app authors do extra translation work. This explores whether we can keep the "I'm building a task system" mental model **and** get the layered composition by injecting the base path at instantiation. Sparring doc.*

---

## The inversion

Today a protocol module hard-codes its **scheme** and lets the operator wire any **backend** behind it:

```ts
// hard-coded scheme, flexible backend
export const programs = { "task://t/": ... };
export const handlers = { ... };
```

What if we flip which axis is fixed: the protocol module hard-codes its **shape** (path structure, programs, handlers, validators), and the deployer chooses the **base path** at construction time?

```ts
export function taskwatchProtocol(basePath: string): {
  programs: Programs;
  handlers: Handlers;
  uri: { task(id): string; update(id, seq): string; ... };
}
```

The factory builds programs keyed at `${basePath}<sub-prefix>` and URI helpers prepend `basePath`. The same protocol now mounts anywhere:

- `taskwatchProtocol("task://t/")` — legacy standalone.
- `taskwatchProtocol("signed://0xabc.../taskwatch/")` — mounted inside a `signed://` behavioral scheme owned by a pubkey.
- `taskwatchProtocol("immutable://acme-corp/tasks/")` — org-namespaced immutable surface.
- `taskwatchProtocol("local://my-app/tasks/")` — experiment scratchpad.

The behavior the handlers enforce is identical. The classification key is identical **relative to** the base path. The behavior **on top of** the base path is whatever the host scheme of the base path itself guarantees — and that's the layering you want, expressed at deploy time rather than baked into the protocol.

## Why this gives both

- **Cake**: app authors still think in their domain. They write "tasks have updates," not "actually you want immutable signed mutable encrypted." The protocol module's API looks the same; only the constructor takes one extra arg.
- **Eat too**: deployers compose. The protocol can ride on top of `signed://<pubkey>/` and inherit signature enforcement *for free* — taskwatch's protocol doesn't need to know about signing, because the program registered at `signed://<pubkey>/` validates before taskwatch's program even gets called (assuming the rig handles nested programs).

Same trick HTTP frameworks pulled with mount points (`app.use("/api/v1", router)`), pushed into the rig.

## Where it gets sharp

1. **Program composition / chaining.** If both `signed://0xabc.../` and `signed://0xabc.../taskwatch/` have programs, what runs? B3nd today is "longest-prefix wins, one program classifies." If we want the outer scheme's program to enforce its rule *and then* hand off to the inner protocol, the rig either needs program chaining or the outer program returns a decompose code whose handler emits the inner tuple (current pattern). **This is the load-bearing question — needs verification against `b3nd-core` before we go further.**

2. **Base-path validation.** The factory must verify the base path is well-formed (ends with `/`, scheme+host present, no `?`/`#`). And it has to refuse some bases — you can't mount taskwatch under `hash://sha256/` because hash URIs are content-derived and taskwatch writes mutable index docs.

3. **Locators.** Read locators (`task://t/list?status=active`) get rewritten to `${basePath}list?status=active`. Fine for the factory's emitted helpers, but if any caller hard-codes a locator string the abstraction leaks. URI helpers have to be the only way to construct paths — which is already the right discipline.

4. **Discoverability for operators.** Domain-named schemes are self-describing in the route table (`task://**` → "that's tasks"). Injected base paths are more opaque — `signed://0xabc.../taskwatch/` requires more context to read. Probably solvable with a manifest the protocol exposes (`{ name: "taskwatch", basePath, programKeys: [...] }`) so a node can introspect.

5. **The empty-base-path case.** Can a protocol mount at *just* a scheme — `taskwatchProtocol("custom://")`? Yes, that's the legacy mode. The factory should default to nothing — force the caller to pick — so people don't slip back into hard-coded schemes by accident.

6. **Multi-tenancy falls out.** Two taskwatch instances on one rig — `signed://alice/tasks/` and `signed://bob/tasks/` — with independent state. The base path *is* the tenant key. Free as long as the storage backend keys on URI prefix (which most do).

## The two ideas are independent

The thing to keep distinct:

- **Base-path injection** says: *the protocol doesn't know its scheme; the deployer does.*
- **Behavior-named schemes** says: *the scheme should name a behavior, not a domain.*

Either works without the other:

- Inject base paths into `info-rig` and let me mount it at `info://` (a domain scheme) — fine, but no behavior layering happens.
- Define `signed://`, `immutable://`, etc. without any base-path injection — protocols still hard-code full prefixes, but at least they pick behavioral ones.

The combination is where the magic is: behavior-named outer schemes **plus** protocols that mount under them. Worth keeping the two ideas distinct in any doc, because they enable each other but neither requires the other.

## The small experiment to validate

Pick taskwatch (smaller surface than info-rig). Convert `protocol.ts` to take `basePath` as a parameter to a factory; thread it through `TASK_PREFIX`, `taskUri`, `updateUri`, `resourceUri`, `parseUri`. Keep the existing usage as `taskwatchProtocol("task://t/")` so nothing breaks. Then try mounting a second instance under a different base path in a test and see whether the rig handles two parallel program sets cleanly. That's the proof.

## Gating questions before doing any of this

1. **How does the rig resolve overlapping prefixes today?** Read `b3nd-core` and find out. If it doesn't already handle nested programs cleanly, the whole composition story is framework work, not a protocol-author convention.
2. **Does B3nd want to be the framework that pushes mount-points into the protocol surface?** It feels right — uniform PIN, locator opacity, "the rig is the harness" — but it's a real opinion to bake into the skill.
3. **Is the manifest pattern (protocol exposes its `programKeys` for introspection) worth it on its own merits**, separate from base-path injection? It might be a useful operator-side primitive regardless.
