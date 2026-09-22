# Networking

Authority: protocol, identity, lifecycle, and safety requirements live in
`docs/mvp-contract.md`. Exact v1 DTOs and byte fixtures are frozen with the
Phase 02 implementation.

## Model

The server is authoritative from the first playable slice. Clients send intent;
the server validates identity and current state, advances headless rules, and
publishes recipient-specific views, events, and results.

The server owns:

- lobby/session lifecycle and ownership;
- positions, collision, interactions, and story progression;
- dice, combat, inventory, death, and run completion;
- input ordering, revisions, event IDs, deadlines, and reconnect; and
- every decision that can affect another player.

The client owns rendering, input collection, local UI/presentation state, and
audio playback after an authoritative event.

## Movement Presentation

The server simulates at `20 Hz`; the client renders at `60 FPS`. The client
predicts its own character from local input and reconciles to each
authoritative position, replaying unacknowledged inputs. Remote characters are
interpolated between authoritative snapshots with a small fixed render delay.
Prediction never triggers interactions, story events, or collision outcomes.
Phase 03 implements this and Phase 10 measures how it feels across regions.

## Early Vertical Slice

Phase 02 proves the complete path before broad gameplay:

```text
client intent
    -> WebSocket DTO
    -> session owner
    -> headless game_core
    -> revision + event/result
    -> recipient view
    -> both clients converge
```

The slice covers lobby create/join, one interaction, one dice check, one legal
combat action, one rejected action, and summary. Later behavior extends this
path; it is not migrated from client authority.

## Transport

WebSockets fit the small co-op update model and the `axum`/`tokio` server.
Production uses WSS through trusted environment endpoints. Steam lobby metadata
contains discovery/compatibility data and the session's region ID from the
trusted allowlist, never arbitrary endpoints or authority.

At session creation, the creator's client probes the trusted region endpoints
and creates the session in the lowest-latency region. Joiners and rejoiners read
the region ID from the Steam lobby and connect to that region. Sessions never
migrate between regions or processes.

JSON v1 prioritizes debuggability. Before admission, authentication,
create/join, and rejoin use the pre-session envelope defined by the contract.
After identity/seat binding, session messages add server-issued session/player
IDs. Every envelope carries a type, protocol version, per-direction transport
sequence, and typed payload. Gameplay inputs also carry a per-player
order number retained across reconnect and a based-on revision.

Protocol rules:

- DTO direction and unknown-field behavior are explicit.
- Production supports one gameplay protocol version. Unsupported clients receive
  `update_required` before admission; deployments drain admitted sessions before
  removing that server version.
- Create/join/rejoin carry all five aggregate content identity fields. Match
  them exactly before seat allocation, connection takeover, or gameplay state
  disclosure. `content_mismatch` preserves the existing reservation/connection;
  it cannot grant authority or trigger a content download. Identity fields and
  rejection fixtures start in Phase 02; canonical content fixtures follow in
  Phase 04.
- IDs are opaque and server-generated.
- Every admitted expected input receives exactly one result.
- Recipient projections expose no secret or other-player private state.
- State is coalescible; semantic events/results are not silently discarded.
- Malformed, unauthorized, stale, duplicate, gap, rate, queue, and service
  failures have stable bounded behavior.
- Every message, ledger, mailbox, writer queue, timeout, retry, and fan-out is
  bounded.

Internal queue capacities are implementation budgets justified by tests and
measurements. They are not copied into protocol documentation unless a client
must know them.

## Identity

Production create/join/rejoin validates a fresh Steam proof for the expected app
and ownership through the Steam Web API. The server issues all session/player
IDs.

- Joining an existing session also requires the contract's server-held join
  grant for the exact identity/session, authorized by the current connected
  lobby owner. Steam metadata and an applicant's membership claim cannot
  substitute for that grant.
  Allocation consumes it atomically; rejected admission leaves it unconsumed.
  Phase 02 proves this with local identities; Phase 07 connects owner approvals
  to the Steam invite/membership flow before game admission.
- Raw Steam tickets are never logged or persisted.
- A returning Steam identity may reclaim only its own reserved in-memory seat,
  without needing a new join grant.
- One player has at most one authoritative connection.
- The client acknowledges recipient-specific resync before new gameplay input.
- Stale takeover-close notifications cannot disconnect the replacement.
- The client stores no rejoin bearer token locally.

A local identity adapter supports deterministic development and tests before
Steam staging is available. It cannot compile into release features/packages.

## Lifecycle And Recovery

Session states, departures, pause/resume, grace and lifetime windows, and
deployment drain are defined in the contract's
[session lifecycle](mvp-contract.md#session-lifecycle) and
[drain](mvp-contract.md#in-memory-sessions-and-draining) sections. In short:
lobbies are private or friends-only and invite-based; disconnected seats are
reserved through bounded grace; session state lives only in the owning process;
drain refuses new admission and run starts, ends idle lobbies, and lets active
runs finish through summary. Maintenance never migrates session state, so
permitted rejoin traffic must reach the owning process while it lives.

## Slow And Failed Peers

State updates are latest-value coalesced. Required semantic events/results use
bounded ordered bundles. A client that cannot drain its bounded writer budget is
disconnected without allowing unbounded memory growth or stalling healthy
sessions.

Heartbeats detect dead connections. Rate limits and pre-auth concurrency bounds
protect expensive identity validation. Trusted proxy configuration, not
user-supplied forwarding headers, determines source attribution.

## Audio

The server never streams game audio. It emits semantic cue events, and each
client plays its matching local asset. This keeps audio out of transport
authority and bandwidth budgets.
