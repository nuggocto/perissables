# Progress Tracker

This file tracks phase gates, not day-to-day tasks. Active implementation work,
owners, and small subtasks belong in issues or a project board. A phase checkbox
links its relevant code, CI, test, QA, security, benchmark, or decision evidence.

`docs/mvp-contract.md` owns product and compatibility requirements.
`docs/roadmap.md` owns execution order.

## Phase 00 - Scope Freeze

- [x] Product scope, non-goals, authority model, and milestones are clear.
- [x] Documentation authority and repository boundaries are clear.
- [x] Evidence-gated decisions are distinguished from locked contracts.
- [x] Define gameplay, admission, lifetime, and drain rules in the
  [contract](mvp-contract.md), with focused [acceptance cases](test-strategy.md#required-high-risk-coverage).
- [x] Documentation audit corrections recorded: shared loot closure, owner-only
  start, lifetime renewal, validation layers, support authority, QA verdicts,
  and target-cadence performance gates. Implementation/evidence remain unchecked.
- [x] Phase 00 complete.

## Phase 01 - Repository Bootstrap

- [ ] Create the six-member Rust workspace with resolver 3, add the root
  `rust-toolchain.toml`, and commit `Cargo.lock`.
- [ ] Add minimal client/server entrypoints, shared logging, local `mise`
  pins for tools outside the Rust toolchain, local tasks, and the locked CI
  checks.
- [ ] Pin reproducible Linux/Windows native dependency acquisition, select the
  exact Steam Linux Runtime/container and minimum host baseline, and compile
  both minimal targets.
- [ ] Compile disposable client authentication-ticket plus lobby/invite API
  spikes on both targets against Steam's public test app; validate a ticket
  from a server stub through the Steam Web API.
- [ ] Record the approved numeric monthly hosting ceiling and cost assumptions;
  check Railway plan/region availability for the candidate topology.
- [ ] Run the Railway drain spike: record whether a draining deployment still
  receives new WebSocket connections, existing-socket behavior, and the maximum
  draining time; revise the drain/rejoin contract before Phase 02 if it fails.
- [ ] Keep the versioned built-in content schema and fixtures in this
  repository.
- [ ] Phase 01 complete.

## Parallel track - Steamworks access

- [ ] Obtain Steamworks partner access and pay the app fee (needed before the
  Phase 06 store-presence step and Phase 07; release requires `30` days after
  the fee).

## Phase 02 - Early Authoritative Vertical Slice

- [ ] Two local test identities create/join one lobby through the real protocol
  and current-owner join grants.
- [ ] One map interaction, dice check, legal combat action, and rejected action
  resolve on the server.
- [ ] Both clients converge on the same summary revisions/events.
- [ ] Prove owner-only start and exclusive party-wide interaction/frozen-world
  behavior using a structurally valid partial aggregate and externalized UI.
- [ ] Freeze content-identity admission DTOs and exact-match/mismatch fixtures;
  mismatches cannot allocate seats or disclose gameplay state.
- [ ] Freeze join-grant DTOs/bounds; prove owner-only issuance, identity/session
  binding, atomic consumption, failure preservation, expiry/revocation, and
  rejection of uninvited joins without weakening reserved-seat rejoin.
- [ ] Release feature checks prove the local identity adapter is absent.
- [ ] Run a headless/internal rules playtest and record unclear check, combat,
  and turn feedback for Phase 05.
- [ ] Phase 02 complete.

## Phase 03 - World Runtime

- [ ] Implement fixed client update/render loops, server-owned cardinal
  movement, local-player prediction/reconciliation, and remote interpolation.
- [ ] Load content through the `content` crate and externalize UI strings.
- [ ] Load one TMX map with collision, bounds, camera, and interaction behavior.
- [ ] Extend the authoritative transcript with movement and interaction
  boundary/error coverage.
- [ ] Phase 03 complete.

## Phase 04 - Story Schema And Runtime

- [ ] Implement built-in schema v1 and its strict repository-owned
  validator/conformance corpus, separate from release-completeness checks.
- [ ] Implement declarative story nodes, choices, checks, effects, encounters,
  and bounded transitions.
- [ ] Freeze parser/resource ceilings from typical, large, limit, and rejected
  fixtures.
- [ ] Pass abstention/default, discarded/replaced ballot, early close,
  tie/deadline, dialogue advance, and story-check actor/wait/cancellation
  transcripts; validate vote defaults and
  canonical identity/checksum fixtures.
- [ ] Phase 04 complete.

## Phase 05 - Combat, Characters, Death, And Loot

- [ ] Complete roster, dice, combat, spell/item, death, loot, victory, and wipe
  behavior through authoritative state machines.
- [ ] Add deterministic public transcripts for legal, rejected, boundary, and
  rollback behavior.
- [ ] Qualify bounded effect replacement, check-modifier consumption, item use
  without rolls, full-inventory grants, shared loot closure/ordered assignments,
  HP/death persistence, charge reset, and round-limit loss.
- [ ] Remove every temporary debug character/action from release features.
- [ ] Run a blind combat playtest outside the implementation team and record
  turn clarity, idle time, encounter length, rules questions, and desire to
  replay.
- [ ] Phase 05 complete.

## Phase 06 - Complete Run Loop And Presentation

- [ ] Complete story selection, ready/start, summary/reset, and three repeated
  runs without stale state.
- [ ] Ship the supermarket presentation and storage-room area, one scalable
  keyboard-operable UI, settings, and fallback behavior.
- [ ] Complete event-driven ambience, music, and SFX without voice.
- [ ] Qualify leave/expiry in each session state, vacant leadership, pause/resume,
  last-player continuation, wipe/end precedence, two-player minimum on reset,
  the absolute summary timeout, and lifetime renewal on lobby entry/valid start.
- [ ] Pass near-expiry start, pause-boundary, and failed-settings-write transcripts.
- [ ] Publish the Steam "Coming Soon" page and the public site with real
  screenshots after the initial site publication checklist passes.
- [ ] Run the first complete story as a blind playtest and resolve blockers in
  comprehension, party downtime, run length, or willingness to replay.
- [ ] Phase 06 complete.

## Phase 07 - Multiplayer And Identity Hardening

- [ ] Add authorized staging Steam validation plus end-to-end lobby creation,
  join, and invites; prove expected app/ownership binding and owner-grant
  authorization before game admission, including owner handoff.
- [ ] Reject direct uninvited joins and stale/wrong-principal grants with no
  seat allocation, gameplay disclosure, or reservation mutation.
- [ ] Qualify ownership-sync failure, removed-member re-grant suppression,
  real-provider ticket lifecycle/replay policy, and fail-closed provider errors.
- [ ] Implement fresh-ticket reserved-seat reclaim, takeover, acknowledged
  resync, heartbeat, replay rejection, and stable public errors.
- [ ] Qualify bounded rate, mailbox/writer, slow-client, and transport-chaos
  behavior.
- [ ] Qualify expiry-before-rejoin, ballot discard across reconnect, gameplay
  pause with only dead spectators connected, and mismatched takeover preservation.
- [ ] Record the small authorized Steam test-account pool needed for identity
  qualification, separate from synthetic isolated capacity identities.
- [ ] Phase 07 complete.

## Phase 08 - Built-in content production

- [ ] Complete the [release inventory](mvp-contract.md#release-content-minimum)
  and pass graph/transcript checks for supported surviving routes.
- [ ] Record written rights/provenance for assets produced by the owner and
  credited friends plus any purchased sources; complete English text and
  fluent collaborator-reviewed French text.
- [ ] Phase 08 complete.

## Phase 09 - Playable MVP Stabilization

- [ ] Deterministic 2-, 3-, and 4-client full runs converge.
- [ ] Repeated run, reconnect, capacity, slow-client, settings, supermarket
  presentation, and single-UI regression scenarios pass.
- [ ] No blocker/critical defect or unexplained intermittent result remains.
- [ ] Phase 09 complete: Playable MVP.

## Phase 10 - Regional deployment and drain decision

- [ ] Freeze representative regional latency, full-run, reconnect, drain,
  forced-stop, and rollback workloads from the implemented game.
- [ ] Measure candidate Railway regions, creator-region session placement,
  and owning-process routing on production-shaped
  infrastructure; freeze acceptable cross-area latency with controlled
  impairment playtests.
- [ ] Record latency by source/host region, drain duration, sessions
  completed/lost, deployment time, CPU/RSS/network, and operator steps.
- [ ] Prove idle-lobby closure, rejected run starts, summary-to-end transitions,
  and natural drain with connected clients under absolute summary/drain deadlines.
- [ ] Freeze the smallest Railway-only region set within the approved hosting ceiling,
  creator-region placement/rejoin routing, drain deadline, run-lost
  behavior, and rollback procedure.
- [ ] Phase 10 complete.

## Phase 11 - Regional service and graceful drain

- [ ] Implement the selected regional placement and owning-process routing.
- [ ] Refuse admission, grants, and run starts during drain; end idle lobbies
  and completed summaries while preserving eligible running/summary rejoin.
- [ ] Prove bounded cleanup exits with clients still connected; summary timeout
  and rejoin cannot extend drain. Distinguish maintenance closure from run loss.
- [ ] Pass production-shaped regional loss, process replacement, forced-stop,
  crash/run-loss, observability, and rollback tests.
- [ ] Phase 11 complete.

## Phase 12 - Release QA, Balance, And Performance

- [ ] Freeze the reference client machine, staging shape, benchmark workloads,
  QA OS builds/UI scales, `60 Hz` reference mode, timer/driver noise floor, and
  target-cadence release gates.
- [ ] Freeze the artifact matrix and authorized-account small-load checks from
  `docs/benchmark-plan.md`.
- [ ] Complete consented playtests and report onboarding/run timing with
  abandonment and uncertainty.
- [ ] Run the pre-release QA matrix and calibrated client/server benchmarks on
  production-profile candidate digests.
- [ ] Qualify Steam authentication with the authorized account pool and run
  capacity tests with the isolated non-release synthetic identity adapter.
- [ ] Prepare and assign owners for Steam store media/copy, age/content
  disclosures, launch languages, pricing, privacy/support contacts, incident
  handling, server operating cost, and shutdown policy.
- [ ] Phase 12 complete.

## Phase 13 - Steam Packaging And Production

- [ ] Build reproducible Linux/Windows packages and sign Windows artifacts
  through the protected signing workflow.
- [ ] Verify the Phase 07 Steam lobby/invite flow against final depots; complete
  production deployment, observability, and the rollback drill.
- [ ] Rerun final QA, client frame gates, and bounded server performance/lifecycle
  checks against exact production digests using the authorized Steam account pool.
- [ ] Record an eligible final QA verdict and explicit acceptance of any known
  non-blocking issues; required journeys/criteria cannot be waived.
- [ ] Rerun capacity gates on the separate production-profile adapter build from
  the same final revision; record both digests, the differences note, and
  production adapter-absence proof.
- [ ] Approve and publish store materials, age/content disclosures, pricing,
  launch languages, end-user terms, privacy/support contacts, third-party
  notices, contribution-policy decision, asset provenance, the final public
  site content, and the operating plan.
- [ ] Phase 13 complete: Release-ready.

## Post-MVP

Track additional themes/UI, voice, durable active runs, normal solo play,
achievements, additional locales, and additional platforms in separate post-MVP backlogs. They are
not Phase 01-13 completion gates.
