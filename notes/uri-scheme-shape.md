# URI scheme shape — domain-named vs. behavior-named protocols

*A critique of a pattern I keep seeing in the protocols we ship, with options for how the skill should teach it. Sparring doc, not a decision.*

---

## What I'm seeing

The last two B3nd protocols we built picked schemes that name **the kind of thing being stored** rather than **the rule the protocol enforces**:

- `bizdev/tools/info-rig` writes `info://<typeDir>/<theme>/<slug>` — a scheme called `info`, owning everything that smells like bizdev information.
- `taskwatch` writes `task://t/<id>`, `task://t/<id>/u/<seq>`, `task://t/list?...` — a scheme called `task`, owning the whole task domain.

Both are coherent. Both work. But both are saying *"I am the protocol of tasks / information"* — the scheme is the **domain**. That's the pattern I want to question.

Compare to schemes that B3nd's own conceptual docs lean on as examples in PROTOCOL.md:

- `hash://sha256/<hex>` — content-addressed. The scheme names a **property of the URI/payload relationship** (the URI is the hash of the bytes), not what the bytes mean.
- `immutable://...` — write-once. The scheme names a **rule**: once a value exists, refuse.
- `signed://<pubkey>/...` — pubkey-gated. The scheme names a **trust posture**: writes require a signature by the key that owns the prefix.

These last three describe **behavior the program enforces**. The first two describe **what kind of data is inside**. That's the axis the critique is on.

---

## Two concrete quirks domain-named schemes generate

### 1. The `scheme://<id>` shape isn't classifiable

B3nd programs are keyed by **URI prefix** and the URI grammar is `scheme://host/path`. The `host` segment is part of the prefix the routing layer matches on. So:

- `task://abc123` puts the id in the host slot. Every task is its own host. You can't write `programs["task://abc123"]` — and you don't want to, that's per-task. A single `programs["task://"]` works, but at that point the program has no useful structure to classify against beyond "the whole scheme is mine."

Taskwatch already worked around this by inserting a `t/` hop: `task://t/<id>`. That gives `task://t/` as a stable prefix, and the id is back in the path where it can be parsed instead of consumed as routing metadata. The workaround is fine — but it's a workaround for a thing the URI grammar quietly tells you: **don't put opaque ids in the host position**. Worth saying out loud in the skill.

(Cf. `taskwatch/src/protocol.ts:13`, `TASK_PREFIX = TASK_SCHEME + TASK_PATH`. The `t/` exists for exactly this reason.)

### 2. Proliferation of foundational surfaces

If every domain gets its own scheme, the rig's route table grows linearly with apps:

```ts
read: [
  connection(infoClient, ["info://**"]),
  connection(taskClient, ["task://**"]),
  connection(noteClient, ["note://**"]),
  connection(recipeClient, ["recipe://**"]),
  // ...
]
```

Each scheme arrives carrying its own:

- Trust model decision (open? pubkey-gated? operator-signed?).
- Listing/index convention (`?fn=ls`, `task://t/list?...`, etc.).
- Content-vs-metadata split (hashed body + indexed meta).
- Immutability story.

These are the **same problems re-solved** in protocol-specific vocabulary, every time. The protocol designer reinvents `hash://`, reinvents `signed://`, reinvents `immutable://` — but flavored to their domain. The operator now has N foundational surfaces to wire and reason about instead of a small set of well-known ones with app data living inside their paths.

The drag isn't visible in the first or second protocol. It compounds.

---

## The alternative: behavior-named schemes

What if the scheme names the **rule**, and the application domain lives inside the path?

| Behavior-named scheme | What program enforces | App-domain examples (inside path) |
| --- | --- | --- |
| `hash://sha256/<hex>` | URI must be SHA-256 of payload | `hash://sha256/abc...` (no app namespace; content is global) |
| `immutable://<ns>/<path>` | Refuse if URI already has a value | `immutable://bizdev/decisions/hackathons/community-machine` |
| `signed://<pubkey>/<path>` | Payload must be signed by `<pubkey>` | `signed://0xabc.../taskwatch/t/abc123` |
| `mutable://<pubkey>/<path>` | Latest signed write wins | `mutable://0xabc.../profile` |
| `encrypted://<ns>/<path>` | Payload is opaque ciphertext; nodes never decode | `encrypted://0xabc.../inbox/2026-06-17/...` |

Now:

- A protocol designer building "info" picks `immutable://bizdev/...` instead of inventing `info://`. They don't redefine immutability — they inherit it.
- A protocol designer building "tasks" picks `mutable://<owner>/taskwatch/...` for the index and `hash://sha256/...` for bodies — both already well-defined surfaces.
- The rig has a small, stable set of foundational schemes: `hash://`, `immutable://`, `signed://`, `mutable://`, `encrypted://`. App namespaces live one path segment in.
- Operators wire backends to *behaviors*, not to apps. "Immutable goes to S3 with object-lock, mutable goes to Postgres, hash goes to IPFS" — that decision is the same for every app on top.

This is also already what PROTOCOL.md hints at in "URI conventions":

> URIs express *behavior*, not meaning. The prefix selects the program; the suffix is whatever the protocol wants.

The skill *says* this. The protocols we're writing don't *do* it. That's the gap.

---

## Counter-arguments (where the critique is weaker than it looks)

I want to make the steel-man case before recommending anything.

**1. "Behavior is enforced by the program, not by the scheme name."**

True. A `task://` scheme could enforce immutability inside its handler just fine. The scheme is just a routing key — naming it after a behavior is a *convention*, not a guarantee. Someone writing `immutable://` could still register a program that mutates. So the win is in **shared mental model and reusable program code**, not in any framework-enforced semantics.

That's still a win, but it's a softer one than "the framework will stop you."

**2. Domain-named schemes have routing/storage benefits.**

`info://**` lines up cleanly with one backend, one storage shape, one listing convention. The "infrastructure operator" experience is *simpler* when each app gets its own scheme — you don't have to mux multiple apps through `mutable://` and figure out which path-prefix goes to which Postgres database.

The behavior-named approach trades single-protocol clarity for ecosystem-level convergence. Worth being honest that there's a real tradeoff.

**3. App authors think in their domain, not in behaviors.**

A bizdev tool author wants to write "an info system." Telling them "actually you want an immutable pubkey-namespaced thing" is a layer of indirection they may not want. The skill would have to teach this translation explicitly, or it'll feel like architecture astronautics.

**4. Maybe the real lesson is narrower than the framing suggests.**

The two observed quirks have two different root causes:

- **`scheme://<id>` not working** is a URI-grammar rule. Fix: don't put opaque ids in host slot, regardless of whether the scheme is domain- or behavior-named.
- **Surface proliferation** is a cultural/library issue. Fix: ship a small set of canonical behavior-named protocols in `b3nd-canon` (or elsewhere) and teach app authors to compose them.

These are independent. We could fix the first with a paragraph in PROTOCOL.md and leave the second open. Or we could go bigger.

---

## Options for the skill

In rough order of ambition:

### Option A — Just teach the `scheme://<id>` gotcha

Add a paragraph to PROTOCOL.md (under "URI conventions"):

> **Don't put an opaque id directly after the scheme.** `thing://abc123` puts `abc123` in the URI's *host* slot — the part program prefixes match on. A program keyed at `thing://abc123` wouldn't generalize; a program keyed at `thing://` has no further structure to classify. Insert at least one stable path segment so the routable prefix is meaningful: `thing://t/abc123`, `thing://by-id/abc123`, etc. This also keeps listing locators (`thing://t/list?...`) inside the same routable prefix as the items they index.

This is uncontroversial and would have prevented one of the taskwatch quirks. Low cost, narrow scope.

### Option B — Teach "scheme = behavior, path = domain" as the preferred shape

Add a short section to PROTOCOL.md (or VISION.md, alongside "URIs vs locators"):

> **Scheme names a behavior; path names the application domain.**
>
> The framework keys programs by URI prefix and routes by URI pattern. That makes the scheme a *behavioral* knob, not a *thematic* one. Prefer schemes that name what the program enforces — `immutable://`, `signed://<pubkey>/`, `hash://sha256/`, `encrypted://` — and put the application's domain inside the path: `immutable://bizdev/decisions/...`, `signed://<pubkey>/taskwatch/t/...`.
>
> Two reasons:
>
> 1. Apps composing on top of B3nd benefit from a small set of well-understood foundational schemes. A "decisions log" and a "task index" both want immutability; they shouldn't reinvent the rule.
> 2. Behavior-named schemes naturally avoid the `scheme://<id>` trap: there's always at least one structural segment between the scheme and the opaque part.
>
> Domain-named schemes (`info://`, `task://`) are not wrong — they're just one app's worth of foundational surface, written from scratch. If your protocol is genuinely sui generis (no existing behavioral scheme captures it), invent a new scheme. Otherwise, compose.

Plus an example pair: how `info-rig` and `taskwatch` would look mapped onto behavior-named schemes.

### Option C — Ship the canonical behavior-named protocols

Actually publish `immutable://`, `signed://`, `mutable://`, `encrypted://` (and the existing `hash://`) as a small set of well-defined B3nd protocols that anyone can install and compose. The skill then *recommends* composing them. This is the largest move and is bigger than a skill change — it's a library decision.

Where would they live? `b3nd-canon` is the natural home (it already ships `hash://sha256/` and envelope/encryption primitives), but TARGETS.md flags canon as "very early." So this is a multi-step thing: stabilize canon → publish the standard protocols → teach the skill to recommend them.

---

## My current take

I'd start with **A + B** as a skill change and treat **C** as a separate planning conversation for `b3nd-canon`.

A on its own is a clear win (the gotcha is real and concrete). B is the soft framing change — naming a value, not adding a constraint — and I think it'll change how the next protocol gets written without requiring any library work. C is the most leveraged in the long run, but it depends on canon maturing, and shipping a "standard protocol library" prematurely freezes choices we may still want to make differently.

The thing I'm least sure about is **whether B over-indexes on a one-protocol-fits-many-apps worldview**. There's a real case that protocol-per-domain is healthy — each app gets to choose its trust model, its listing grammar, its content/meta split. The B framing nudges toward consolidation; it may be the wrong nudge for a framework whose whole pitch is "you define your own rules."

Worth talking through with you before I write any skill edits.

---

## Specific questions for you

1. Do you want the skill to **push** authors toward behavior-named schemes (B), or just **warn** about the `scheme://<id>` shape (A)?
2. If B, is the implicit goal *eventually* C — a canonical set of behavior-named protocols — or do you want app authors to keep rolling their own, just with better naming?
3. Are the existing `info://` and `task://` protocols staying as-is, or do you want to migrate them as part of this conversation? (My instinct: leave them, treat them as Pre-Pattern-Discovery, capture the new pattern for the next protocol.)
4. Where should the "scheme = behavior" rule live — `PROTOCOL.md`'s "URI conventions" section, or hoisted up to `VISION.md` so even app developers see it?
