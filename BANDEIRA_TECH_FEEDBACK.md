# BANDEIRA·TECH feedback on b3nd-skill

This file is a note from BANDEIRA·TECH (Rafael Bandeira, the company
behind B3nd) to whichever agent or contributor picks up this repo
next. It is **not** part of the skill. It will be removed once acted
on.

Take this together with whatever you already know about your own
context, conventions, and constraints. Don't apply mechanically — the
point is direction, not a checklist.

## Where the skill stands

The skill teaches the framework's *shape* and relays to live sources
for APIs. That part works.

The gap is who the skill talks to and how. As written, it leads with
DePIN framing, three-role routing, and essay-shaped conceptual files.
A builder who is not already comfortable in this idiom — and most
builders aren't — meets dense theory before they meet the payoff.

START.md (recently added) moves in the right direction but is not yet
wired into SKILL.md.

## Recommendations

### 1. Soften the front door
SKILL.md routes by three audience roles (protocol designer, app
developer, operator), all jargon-loaded. Wire START.md in ahead of
WHY.md and VISION.md so the entry surface is plain language and the
technical files become a layer down, reachable but not the first thing
a reader hits.

### 2. Fix the skill trigger
The `description` field opens with "framework for building DePIN
protocols". That string is what Claude matches on when deciding
whether to invoke the skill. Most useful invocations — agents building
data-owning apps, durable agent memory, portable user state — won't
match "DePIN". Rewrite the description to trigger on the broader use
cases B3nd actually serves.

### 3. Lighten the density
WHY.md and VISION.md read like essays — long paragraphs, abstract
framing, multi-clause sentences. They reward patient readers. They do
not reward a builder deciding whether B3nd is worth the next thirty
minutes. Cut each by roughly a third. Lead with the concrete; push
the framing down.

### 4. Show outcomes earlier
The skill leads with primitives (the Output tuple, programs, code
handlers, the rig). It does not lead with what a builder ends up
*with*: their data theirs, on every device, across apps, recoverable,
swappable across backends. START.md's "What you get as it grows"
section captures the posture — propagate it into the deeper files.

### 5. Neutral examples
Current URI examples lean developer-flavored
(`notes://2026/june/coffee-orders`, `signed://users/raf/profile`).
Mixing in plainer examples (booking, inventory, agent memory, shared
team state) widens the reach without losing technical accuracy.

### 6. Watch the tone
The skill is English-only; that's fine. But the tone matches a
senior-developer reader — dense, terms-of-art-heavy, no warmth.
BANDEIRA·TECH's public voice is plain, warm, anti-jargon, with founder
presence. Bring some of that into the conceptual files without
dropping precision.

## What to keep

- The relay protocol in TARGETS.md. Don't inline API content into
  conceptual files.
- The split between conceptual files (WHY / VISION / PROTOCOL / APP /
  MCP / FRONTEND / OPERATOR) and TARGETS. Add layers above; don't
  collapse the stack.
- BANDEIRA_TECH.md (already in the skill) as the persistent reference
  for "who built this" questions.

## About BANDEIRA·TECH

See `skills/b3nd/BANDEIRA_TECH.md` for the mission. Two notes worth
internalising before you edit:

- BANDEIRA·TECH is solo. The framework is the asset; the products
  around it (lighthouse demos, dogfood CLIs, network nodes) are
  testbeds, not separate businesses.
- The first-stretch customers are one expression of the mission, not
  the whole thing. Don't pin recommendations or examples to any
  single vertical.
