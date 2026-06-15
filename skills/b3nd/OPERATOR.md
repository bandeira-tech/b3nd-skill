# Running a node

Operators run B3nd nodes. A node is a process that:

1. Loads a protocol's schema.
2. Receives messages from clients.
3. Dispatches each message through the schema's programs.
4. Stores accepted messages in one or more backends.
5. Serves reads, lists, and subscriptions.

The framework gives you the dispatch + transport machinery. The operator
picks the backends, the trust posture, and the deployment shape.

## The node mental model

A node is **postmaster**, not author. It does not interpret content. It
runs the protocol's rules and stores what the rules accept.

This means:

- Encrypted payloads pass through opaquely.
- A misbehaving program is the protocol designer's fault, not the node's.
- An operator's job is uptime, durability, and replication — not policy.

## Backends

A node composes storage from one or more backends. Typical choices:

- **Memory** — fastest, no durability. For tests and ephemeral nodes.
- **PostgreSQL** — durable, queryable, transactional. The common default
  for production single-node deployments.
- **MongoDB** — durable, document-shaped. Useful when payloads are
  variable.
- **HTTP** — proxy to another node. Useful for replication and gateways.
- **Object storage (S3-class)** — for large blobs, often paired with a
  metadata backend.

Reads try backends in the order they're composed. Writes broadcast to all
of them. The protocol does not care which backends are present.

## Deployment shapes

- **Single node** — one process, one backend (often Postgres). Smallest
  durable deployment.
- **Replicated** — multiple nodes pointing at peer nodes via HTTP, each
  with its own local backend. Reads can be served locally; writes
  propagate.
- **Managed cluster** — an operator runs a fleet behind a load balancer,
  with shared durable storage.
- **Browser / embedded** — a node-shaped process running inside the
  client. Useful for offline-first apps.

Pick the shape that matches the protocol's trust model and the app's
latency budget.

## Keys and identity for the node

Some protocols require the node itself to have an identity (signing
relayed messages, attesting to replication). Others don't. The framework
provides key generation and storage helpers; how a node identity is
managed is an operator decision.

If a node is acting on behalf of users (managed mode), the operator must
decide how user keys are held — escrowed by the node, signed remotely by
the user, or delegated via capability tokens.

## What to monitor

- Backend health — Postgres connection pool, disk, replication lag.
- Dispatch latency — how long programs take to validate.
- Reject rate per program — sudden spikes suggest protocol abuse or a
  client bug.
- Replication freshness — peer-to-peer lag if running multi-node.

## Before configuring anything

Concrete config keys, env var names, CLI flags, and backend URIs change
across versions. Do not write a docker-compose, Terraform, or wrangler
config from memory. Follow TARGETS.md to find the current operator docs
for the version the user is running.
