# Architecture

Authority: product and compatibility requirements live in
`docs/mvp-contract.md`. This document explains the intended structure and the
reasoning behind it. Regional deployment and measured capacity budgets remain
provisional until their named decision gates pass.

## Architecture Goals

- Exercise the real client/server boundary from the first playable slice.
- Keep all gameplay rules headless, deterministic, and server-authoritative.
- Keep the story, characters, enemies, and supermarket presentation data-driven.
- Prefer small, explicit modules and bounded resources.
- Lock implementation details only after evidence makes the trade-off clear.

## Stack

| Layer | Technology | Responsibility |
| --- | --- | --- |
| Client | Rust + `raylib` (`raylib-rs`) | Window, rendering, input, UI, audio, local movement prediction |
| Server | Rust + `axum` + `tower` + `tokio` | Sessions, authority, health, networking |
| Shared rules | Headless Rust crate | Story, dice, combat, character, and run state machines |
| Protocol | `serde` + `serde_json` over WebSockets | Versioned intents, views, events, and errors |
| Content | JSON + TMX loaded and validated by the `content` crate | Immutable built-in declarative data shared by client and server |
| Session state | Server memory | Active lobbies and runs; no database or long-term save |
| Hosting | Railway | Regional staging and production server environments; Steam Web API ticket validation |
| Distribution | Steam | Ownership, lobbies/invites, and Linux/Windows depots |

## System Shape

```text
[Validated built-in story/character/enemy data]
                  |
                  v
        [Headless game_core]
                  |
                  v
       [Authoritative server]
          |
   versioned views
          |
       WebSocket
          |
          v
            [Client]
     render, input, audio
```

The client sends intent and renders recipient-specific authoritative views. It
never decides movement success, story outcomes, dice, combat, inventory, or run
completion.

## Early Authoritative Vertical Slice

Phase 02 is the first executable product proof. It must use the real client,
server, protocol, and headless rules path for:

1. creating one lobby and joining it with two local test identities;
2. starting one tiny run;
3. moving to and activating one map interaction;
4. resolving one server-generated dice check;
5. resolving one legal and one rejected combat action; and
6. returning both clients to the same summary state.

The local identity adapter exists only for local tests and development builds.
Production features and packages must not compile it. Steam identity replaces
that adapter without changing gameplay authority.

Later phases deepen this slice instead of relocating client-owned gameplay to the
server. A feature is not part of the game until it travels through the same
intent -> authority -> view/event path.

## Crate Boundaries

```text
Cargo.toml
Cargo.lock
rust-toolchain.toml
mise.toml

crates/
  client/
    src/
      main.rs
      app/
      scene/
      render/
      input/
      audio/
  server/
    src/
      main.rs
      startup.rs
      router.rs
      session/
      netcode/
      identity/
      drain.rs
  game_core/
    src/
      game/
      story/
      combat/
      character/
      item/
      dice/
  content/
    src/
      schema/
      load/
      validate/
      checksum.rs
  shared/
    src/
      protocol/
      ids/
      logging.rs
  integration_tests/
    tests/

assets/
stories/
  builtin/
```

Workspace members are `client`, `server`, `game_core`, `content`, `shared`,
and `integration_tests`. Package names use the `les-perissables-` prefix.

Dependency direction is one-way:

- `shared` owns protocol DTOs, IDs, and common logging setup.
- `content` owns built-in content DTOs, JSON/TMX loading from bytes, pure
  validation, and the canonical checksum. It holds no gameplay rules.
- `game_core` depends on `shared` and `content`.
- `server` depends on `shared`, `content`, and `game_core`.
- `client` depends on `shared` and `content` so it can render the map, text,
  and assets; it does not link `game_core` or gameplay rules.
- `integration_tests` may depend on every workspace crate.

Reverse dependencies and cycles are forbidden.

## Headless game_core

`game_core` has no `raylib`, network, filesystem, environment, wall-clock, or
process-global access.

- State changes only through typed intents, ticks, deadlines, and explicit
  start-run entropy.
- Randomness and time are injected.
- Operations return new revisions plus stable events/results.
- Built-in content arrives as validated `content` DTOs; storage I/O stays in
  the server and client.
- Tests can replay the same transcript without a renderer or socket.

## Runtime Ownership

- One session owner serializes mutation for one session.
- Long-lived tasks belong to one supervised startup/task tree.
- Every task, timeout, queue, and blocking operation is bounded.
- Critical tasks are cancelled and joined explicitly; dropped handles may not
  silently detach authoritative work.
- Native `raylib` and client-side Steamworks calls stay behind small safe
  adapters. The server validates Steam tickets through the Steam Web API and
  does not link the Steamworks SDK.
- `unsafe` is forbidden in domain crates. Adapter unsafe blocks require a
  local `SAFETY` explanation and tests for lifetime/thread-affinity contracts.

## In-memory sessions, regions, and drain

One session owner holds one authoritative lobby/run in memory. There is no
persistence adapter or database in MVP. A session lives in the region its
creator's client selected and never moves; each regional service runs one
process, so the region endpoint reaches the owning process.

Deployment drain, run-loss, and rejoin-while-draining behavior are defined in
[the contract](mvp-contract.md#in-memory-sessions-and-draining). The Phase 01
Railway spike and Phase 10 decision establish whether a draining Railway
deployment can still receive rejoins; the architecture changes before Phase 11
if it cannot.

## Engineering Practices

- Phase 01 adds a root `rust-toolchain.toml` that pins Rust `1.97.1` with
  `rustfmt` and Clippy. `mise` will manage only tools outside that Rust
  toolchain and local task aliases.
- Edition 2024, virtual-workspace resolver 3.
- Commit `Cargo.lock`; CI and release builds use `--locked`.
- Prefer the standard library and concrete types until an abstraction has more
  than one useful consumer or implementation.
- Use typed errors at reusable boundaries and contextual errors in executable
  orchestration. Recoverable input, I/O, and dependency failures do not panic.
- Use structured `tracing` fields without secrets or personal data.
- Keep transport, rules, presentation, content validation, and process draining
  separable at their real ownership boundaries; do not create layers solely to
  match a diagram.
- Add tests with the behavior that introduces them. Profile only measured hot
  paths.
- Follow `docs/test-strategy.md`, `docs/qa-plan.md`,
  `docs/security-model.md`, and `docs/benchmark-plan.md` for evidence.
