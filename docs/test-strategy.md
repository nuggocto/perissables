# Test Strategy

Status: Normative automated-test policy
Owner: Sole developer

Product and protocol behavior comes from `docs/mvp-contract.md`. This document
defines how automated tests earn confidence without becoming a second
implementation or a paperwork system.

## Principles

- Test observable behavior through public interfaces.
- One test protects one behavior; table cases may cover several inputs of that
  same behavior.
- Assert concrete outputs and negative space. A rejected action must not mutate
  gameplay state, consume RNG/a turn, spend resources, or emit gameplay events.
  Assert protocol bookkeeping separately: recording a rejection or advancing an
  expected input sequence is allowed. Independently due expiries/ticks are their
  own transitions and must not be attributed to the rejected action.
- Inject time, RNG state, IDs, transport schedules, process-stop signals, and
  identity-provider responses.
- No sleeps for synchronization, wall-clock dependence, shared mutable fixtures,
  order dependence, or retry-until-green.
- Use real deterministic collaborators when cheap. Substitute only the boundary
  that is unsafe, slow, nondeterministic, or external.

## Fail-First Evidence

Regression tests and critical invariants are observed failing for the intended
reason before acceptance. Routine tests require meaningful behavioral assertions,
but no manufactured implementation defect or standalone red/green evidence.

- Regression fixes preserve the failing test name, command, and failure
  signature in the issue or change record.
- Critical protocol, session-drain, security, and content invariants preserve
  equivalent red/green evidence.
- Routine test-first feature work does not require a permanent test-only commit,
  tree identifier, or standalone evidence bundle.
- Pure refactors state why no new behavior test is needed.

## Layers

| Layer | Protects |
| --- | --- |
| Unit/property | Dice, combat, story rules, bounds, and canonicalization |
| Schema/conformance | Accepted/rejected built-in JSON, TMX, paths, and bytes |
| State-machine transcript | Authoritative revisions, events, rejection, reset |
| Protocol contract | DTOs, directions, versions, sequences, projections, errors |
| Multi-client integration | 2/3/4-client convergence, reconnect, replay, capacity |
| Process lifecycle | Admission stop, drain, deadline, forced stop, and run loss |
| Release smoke | Shipped startup and critical user journeys on supported OSes |

Release smoke is automated end-to-end coverage only when it launches the shipped
entrypoint. Internal server/component tests remain integration tests.

## Selecting Cases

Use boundary analysis, not mechanical case multiplication.

- Cover empty/zero, one, a representative value, the meaningful limit, just
  beyond the limit, malformed input, and the credible operational error where
  those cases can change behavior.
- Do not generate limit-minus-one/limit/limit-plus-one cases for every field when
  a shared validator or property states the same invariant once.
- Use exhaustive cases where the state space is deliberately small, such as all
  d100 results.
- Use property tests for broad invariants such as serialization round trips,
  projection secrecy, queue accounting, canonical checksums, and no mutation on
  rejection.
- Keep fixtures minimal. Production-sized fixtures belong only where size or
  concurrency is the behavior under test.

## Required High-Risk Coverage

- Authoritative transcripts cover one legal and one illegal action at every
  gameplay phase, stable ordering, leader/tied votes, corpse-loot votes,
  spell-only enemy kits, each spell result class and critical redirection
  polarity, death/wipe, and three-run reset.
- Protocol tests cover fragmentation, malformed DTOs, wrong direction/version,
  stale/duplicate/gap sequences, recipient-specific projections, queue refusal,
  reconnect handoff, and slow writers.
- Identity tests cover wrong app/identity, stale Steam proof, takeover, seat
  reclaim, expiry, owner-authorized new-seat admission, provider failure, and
  absence of local/benchmark adapters in release features. Real-provider tests
  qualify ticket cancellation/reuse and ownership; injected tests cover failures
  without manufacturing load against Steam.
- Content tests cover duplicate/unknown fields, resource limits, disabled TMX
  external access, graph errors, reference errors, and checksum stability.
  Separate release-inventory and route checks cover the contract's release
  minimum; content estimates cannot substitute for timing playtests.
- Process-lifecycle tests cover admission refusal after unready, reserved-seat
  rejoin while draining, rejected run starts, idle-lobby closure, summary-to-end
  transitions, natural completion with connected peers, deadline expiry, forced
  stop, and stable run-loss behavior.

Add these focused acceptance transcripts with their owning phase. They describe
required behavior; they are not tests already run.

| Phase | Behavior and required oracle |
| --- | --- |
| 02, extended in 04 | An otherwise authorized join with exact five-field content match admits; change each field independently to get `content_mismatch` without seat allocation, grant consumption, or gameplay state. Missing/malformed fields and wrong protocol retain their distinct errors. Phase 04 adds canonical checksum and locale-selection cases. |
| 02, Steam qualification in 07 | A valid identity with known session ID and matching content but no grant gets `join_not_authorized` without seat/state disclosure or reservation mutation. Only the current connected lobby owner can issue/revoke grants. Wrong identity/session, expired/revoked grants, and consumed-grant replay cannot allocate a seat. Successful admission consumes one grant atomically; capacity or content failure does not. Cover grant bounds, owner change, run start, and leave/removal invalidation. Rejoin with a valid reservation needs no new grant. Phase 07 proves the normal Steam invite/membership flow completes owner authorization before game admission, including owner handoff. |
| 02, extended in 06-07 | Only the current connected owner can start; non-owner/stale-owner starts reject without RNG or lifetime extension. Start requires every seat connected, ready, uniquely selected, and any required resync acknowledged. Competing interactions start one party-wide sequence; world positions freeze and movement/interaction input cannot queue effects for the return to exploration. |
| 04 | A structurally valid partial aggregate passes structural validation and fails the separate release-completeness gate. A full inventory with a route bypassing the boss or missing presented counts still fails the route gate. No declared duration makes a timing playtest pass. |
| 04 | Zero story ballots select the declared default; one winner, leader tie, and lexical tie each select the specified choice without RNG consumption. Disconnect discards an existing ballot; rejoin cannot resurrect it. A fresh ballot before closure counts; at the deadline it is rejected. The vote closes early once every connected living player holds a ballot, and a replaced ballot before that point still counts. |
| 04 | Dialogue advances once every connected living player presses continue, or at the `20 second` timeout; a disconnect discards that player's press and no longer blocks advancement; dead spectators neither block nor advance. |
| 04 | Check actors follow eligible leader then lexical fallback even when a different player initiated the interaction. Use unequal stats and a fixed roll that distinguishes them. An empty candidate set leaves the node waiting without RNG/modifier consumption; reconnect resolves once; ending cancels. Disconnect before versus after atomic resolution cannot produce a second roll. |
| 05 | Repeated normal/critical guards replace remaining hit counts; zero damage preserves guard. Repeated bonuses/penalties replace their own slots, combine once, clamp, and both expire on every actual check including criticals. Death/encounter end clears effects; disconnect does not. |
| 05 | Items resolve without RNG, spend exactly one item/turn on legal use, preserve next-check modifiers, and spend nothing on rejection. Full-HP use still spends; full-inventory story grants skip without replacing items or blocking progression. Multiple grants use authored order. |
| 05 | Reaching the combat round limit enters the wipe summary in both a normal and the boss encounter. |
| 05 | No loot ballots or no valid recipient means no assignment. Complete the later corpse's ballots first: no early item assignment occurs while earlier ballots are incomplete. The shared deadline or all-items/all-voters condition closes the phase once. Revalidate each item's recipients after prior assignments; competing awards follow corpse/slot order without overflow or duplicates. Replacing ballots remains possible until phase closure. Defeated-enemy authored loot is distinct from held inventory and draws no RNG; player death and wipe create none. |
| 05 | A normal encounter win retains living HP and inventory, keeps dead players dead, and clears effects. The next encounter restores spell charges but grants no passive healing; the next run restores maximum HP and empty inventory. |
| 06, transport qualification in 07 | Leader expires while all others are disconnected: vacant role, no random draw, then one election on eligible rejoin. Empty lobby ownership similarly recovers. Leave/expiry in each state releases the right seat and removes active characters/items without loot. One-player continuation, no-living-player wipe, no-seat end, and reset to a two-player start requirement are distinct outcomes. |
| 06 | Returning to `Lobby` and committing a valid run start restart the lifetime window. Start just before old expiry gets a full new window; at expiry the session ends before start and cannot be revived. Rejected starts, rejoin, and summary entry do not extend it. Summary returns to a cleared lobby on all-seat acknowledgement or its absolute `2 minute` timeout, capped by lifetime. Story-selection changes clear readiness. |
| 06 | Pre-commit settings write/flush/replace failures preserve the previous usable file; post-commit durability failure keeps the valid committed file and reports uncertainty. Owned temporary files are cleaned or cleanup failure is reported on both OSes. Startup preserves malformed/future files while using defaults in memory. |
| 07 | Due expiry wins over rejoin or a due vote. Dead spectators cannot advance gameplay while every living player is disconnected; gameplay resumes with remaining time, while seat/session/drain expiry keeps running. A content-mismatched takeover neither replaces the connection nor extends grace. |
| 06, transport qualification in 07 | At dialogue, vote/loot, and turn boundaries, process the last living disconnect before progression and verify pause with no RNG or outcome; reverse the serialized order and verify exactly one committed outcome, never rollback or duplicate resolution. Absolute expiry still runs. |
| 07 | Provider timeout/outage, rate refusal, or malformed authentication/ownership response rejects before allocation or takeover, with bounded work and no grant/reservation mutation. Real-provider evidence records newly acquired service-bound tickets, cancellation/reuse, returned player versus license-owner identity, and the frozen replay policy. |
| 07 | Force Steam/server owner disagreement and synchronization failure: Steam callbacks cannot change server authority, automatic grants wait, and existing seats/rejoin survive. A removed player still in the Steam lobby gets no automatic new grant from stale callbacks; a fresh join or explicit owner reauthorization is needed. Owner handoff preserves suppression; its capacity exhaustion refuses automatic grants without dropping records. |
| 10 spike, 11 implementation | Drain with idle, running, and summary sessions. Idle lobbies end immediately; new admission, grants, and queued post-drain `StartRun` are refused. A run committed before drain finishes through summary to `Ended`. Keep peers connected and verify all seats/tasks release and the process exits before its drain deadline. Repeat with missing summary acknowledgements: the absolute summary timeout ends the session, and reconnect cannot extend it. Running/summary rejoin remains valid only before end/expiry. Idle/completed closure shows maintenance; an interrupted run shows run loss. No summary returns to the draining process's lobby. |

Phase 03 movement tests cover prediction reconciliation against injected
authoritative corrections and show that predicted positions never trigger
interactions or events. Delayed/coalesced inputs, oversized claimed time deltas,
and thin collision obstacles must not bypass server-owned speed or collision.
The TMX export fixture preserves collision/event objects without external access.

Phase 13 build checks record the production/capacity-build differences from
`docs/benchmark-plan.md`, including adapter absence in the production artifact.

## Fuzzing

Fuzz network-facing input only: WebSocket framing, protocol decoding, and the
state-machine boundaries it reaches, in bounded isolated processes. Built-in
content is repository-authored and covered by conformance fixtures instead.

- Keep targets narrow and free of external I/O.
- Bound input size, time, memory, recursion, operations, and concurrency.
- Preserve/minimize failures and promote useful cases to deterministic
  regressions.
- A panic, hang, escape, external fetch, unbounded allocation, or cleanup leak
  is a failure.

## CI And Evidence

- Run the pinned commands from `docs/mvp-contract.md` on a clean checkout.
- Run tests in parallel; repeat only concurrency/chaos suites under recorded
  deterministic seeds.
- Preserve the first intermittent failure. The suite remains failing or
  `INCONCLUSIVE` until the cause is understood.
- Coverage percentage is informational. Missing behavior is not excused by a
  high number.
- Phase evidence links CI, important test names, fixture/schema versions, and
  any manual or environment-specific result.
