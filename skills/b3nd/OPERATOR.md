# Running a node

A node is a Rig configured with:

1. A protocol's **programs** and **handlers**.
2. **Storage backends** wired into the receive and read routes.
3. One or more **transport services** exposing the rig to the outside
   world.

The framework gives you the Rig and the PIN interface. The operator
picks the backends, the transports, the trust posture, and the
deployment shape.

## The node mental model

A node is **postmaster**, not author. It does not interpret content.
It runs the protocol's programs and handlers and stores what the
handlers emit.

This means:

- Encrypted payloads pass through opaquely.
- A misbehaving program is the protocol designer's fault, not the
  node's.
- The operator's job is uptime, durability, and replication — not
  policy.

## Storage backends

Backends live in `b3nd-save`. Each backend implements a uniform
`EntityStore` contract and is paired with a `SaveClient` that adapts it
to PIN, so the rig sees it as just another node-shaped client.

Common backends:

- **Memory** — fastest, no durability. Tests and ephemeral nodes.
- **Postgres** — durable, queryable, transactional. The common default
  for production single-node deployments.
- **Mongo** — durable, document-shaped. Good for variable payloads.
- **SQLite** — embedded, single-file, durable. Useful for small nodes
  and offline-first apps.
- **Filesystem / IPFS / S3** — blob-shaped backends, usually paired
  with a metadata backend.
- **Elasticsearch** — query-heavy reads on indexed payloads.
- **LocalStorage / IndexedDB** — browser-side persistent stores.

Reads try wired backends in route order. Receives broadcast to all
matching routes. The protocol does not care which backends are present.

A `SaveClient` reads as "receive payloads X, map them via Y, store on
S." The mapper turns a wire payload into the backend's record shape —
`passThroughRecord` for direct mapping, `mapToBytes` for opaque blobs,
or a custom mapper that decodes envelopes, drops fields, etc.

## Transports

Transport services live in `b3nd-move`. For each supported transport,
`b3nd-move` ships two halves:

- A **service** — incoming bytes → decode → drive the rig → encode →
  outgoing bytes. The server side.
- A **client** — implements PIN, speaks the wire. Drops into a Rig
  route as a regular upstream.

Supported transports: HTTP, WebSocket, gRPC-over-HTTP, MCP. Mix freely.
One rig can be exposed over multiple transports at once; its routes can
be backed by clients speaking entirely different transports upstream.

## Deployment shapes

- **Single node** — one process, one storage backend (often Postgres),
  one or two transports. Smallest durable deployment.
- **Replicated** — multiple nodes, each with local storage, peering
  over HTTP or WebSocket clients in their read/receive routes. Reads
  serve locally; receives propagate.
- **Managed cluster** — operator runs a fleet behind a load balancer,
  shared durable storage, optional dedicated ingest tier.
- **Browser / embedded** — a rig running inside the client with local
  backends (memory/localStorage/IndexedDB) plus remote upstream
  clients. Useful for offline-first apps.

Pick the shape that matches the protocol's trust model and the app's
latency budget.

## Identity for the node

Some protocols require the node to have its own signing identity
(attesting to relay, signing replication batches). Others don't. The
framework provides key generation and storage helpers; how a node
identity is held is an operator decision.

If a node acts on behalf of users (managed mode), the operator must
decide how user keys are held — escrowed by the node, signed remotely
by the user, or delegated via capability tokens. Document the choice.

## What to monitor

- Backend health — connection pool, disk, replication lag.
- Dispatch latency — how long programs and handlers take.
- Reject rate per program code — sudden spikes suggest abuse or a
  client bug.
- Receive throughput per route — fan-out imbalance is a clue something
  upstream is degraded.
- Peer freshness if multi-node — locator-pattern misses across replicas.

## Before configuring anything

Concrete config keys, env vars, CLI flags, and constructor signatures
change across versions. Do not write a docker-compose, Terraform,
Kubernetes, or `wrangler.toml` from memory. Follow TARGETS.md to find
the current operator docs for the version the user is running.
