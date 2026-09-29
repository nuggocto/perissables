# Roadmap

## Planning Rule

This roadmap is an execution order, not a calendar promise. For one developer,
a `4-8 month` release range remains an aspirational hypothesis until the Phase
02 authoritative slice, native target builds, the Phase 01 Railway drain spike,
and the Phase 10 regional deployment decision have produced evidence.

Re-estimate after each of those gates. A failed feasibility gate changes the
plan before dependent implementation begins; it does not get hidden inside a
later phase.

## Milestones

| Milestone | Phases | Outcome |
| --- | --- | --- |
| Authoritative vertical slice | 00-02 | Two clients complete one tiny server-owned run |
| Playable MVP | 00-09 | Complete repeatable 2-4 player game loop with built-in content |
| Release-ready | 00-13 | Regional service, measured performance, Steam packages, and production operations |
| Post-MVP | Separate backlog | Additional themes/UI, durable runs, and other extensions |

## Phase Plan

### Phase 00 - Scope Freeze

Define the player experience, product boundaries, authority model, and completion
evidence. Phase 00 is documentation-only.

Exit: the current contract and documentation map agree, with no implementation
detail labeled final before its evidence gate.

### Phase 01 - Repository Bootstrap

Create the six-member Rust workspace, add the root pinned toolchain, commit the
lockfile, and add minimal client/server entrypoints, logging, CI, legal baseline,
and local development tasks. Keep Rust owned by `rust-toolchain.toml`; use
`mise` only for tools outside the Rust toolchain and local task aliases. Keep the
built-in content schema and fixtures in this repository.

Run only bootstrap-relevant feasibility checks:

- Linux and Windows native dependencies can be acquired reproducibly.
- The exact Steam Linux Runtime/container is pinned and the minimum supported
  Linux x86_64 host requirements are recorded under the support contract.
- Both targets compile minimal client/server shells.
- Minimal client authentication-ticket plus lobby/invite APIs compile on both
  targets in a disposable spike against Steam's public test app, and a server
  stub validates a ticket through the Steam Web API. Record the test app,
  endpoint/key type, and limits; this spike does not qualify production app
  ownership or publisher-key access.
- Railway availability and entry-level subscription limits are checked for an
  initial Americas/Europe/Asia candidate topology; nearby worldwide latency is
  measured later rather than requiring a physical deployment in every area.
  Record the owner-approved numeric monthly hosting ceiling and currency,
  selected plan, and production/staging/traffic assumptions before evaluating it.
- Railway drain spike: deploy a tiny WebSocket server, trigger a replacement
  deployment, and record whether new connections can still reach the draining
  process, how existing WebSockets behave, and the maximum
  `RAILWAY_DEPLOYMENT_DRAINING_SECONDS`. If a draining process cannot receive
  rejoins or lacks a useful window for a full run plus summary, revise the drain
  and rejoin contract before Phase 02. Document forced loss beyond that window;
  the typical run duration is not an upper bound on long or paused runs.
- CI can run the locked Rust checks.

Large Steam account pools, production integration, production signing, and
regional deployment are later gates owned by the phases that need them.

### Parallel track - Steamworks access

Obtain Steamworks partner access and pay the app fee for Les Périssables. This
is administrative work, not a Phase 01 exit condition, but it must finish
before the Phase 06 store-presence step and before Phase 07 needs the real app
ID. Steam requires `30` days between the app fee and release.

### Phase 02 - Early Authoritative Vertical Slice

Build the first runnable product through the real server boundary:

- two local test identities create/join one lobby through owner-authorized
  new-seat admission;
- the server starts one tiny run;
- one map interaction triggers one check;
- one legal and one rejected combat action resolve;
- both clients converge on the same summary.

Freeze the five-field content-identity admission payload, owner-grant DTOs and
bounds, and positive/rejection fixtures against the slice aggregate. Identity,
content, and grant checks precede seat allocation; successful allocation consumes
the grant atomically. Phase 04 adds the full canonical checksum corpus.

Use real protocol DTOs, revisions, events, errors, and bounded queues from the
start, with externalized strings for any player-facing UI. Prove owner-only
start and exclusive party-wide interaction behavior against the partial slice
aggregate. Restart durability, Steam authentication, polished rendering, and broad
content are not required yet. Before broad rendering, run the slice as a
headless/internal rules playtest and record unclear check, combat, and turn
feedback for Phase 05.

### Phase 03 - World Runtime

Implement fixed-timestep client presentation, server-owned cardinal movement,
local-player prediction with reconciliation, remote-character interpolation,
TMX loading/collision through the `content` crate, camera behavior,
interactions, additional externalized UI strings, and the minimal audio event
path. Extend the Phase 02 transcript with collision/export, delayed-input, and
prediction boundary cases rather than creating a separate single-player path.

### Phase 04 - Story Schema And Runtime

Implement the internal built-in schema v1, strict validation, and a declarative
story state machine. Add branching, choices, checks, effects, encounters, and
return/end transitions. Freeze exact parser/resource ceilings with repository
conformance fixtures, not before they exist.
Require the contract's abstention, ballot-discard, early-close, deadline,
dialogue-advance, and story-check actor transcripts before exit. Content
validation rejects missing/invalid vote defaults. Separate structural validation
from release inventory, route checks, and measured playtime; valid partial
aggregates remain usable until Phase 08 completes the release inventory.

### Phase 05 - Combat, Characters, Death, And Loot

Complete the preset roster, dice/combat rules, items, death, corpse loot,
victory/wipe behavior, and deterministic authoritative transcripts. Remove every
temporary debug character and action before exit. Run a small blind combat
playtest with people outside implementation and record turn clarity, idle time,
encounter length, rules questions, and desire to replay before locking the
combat loop.
The exit transcripts also cover guard/modifier replacement, item use without
rolls, full-inventory grants, shared loot-phase closure and ordered assignments,
HP/death persistence, charge reset, and the combat round limit entering the wipe
summary. Resolve the authored enemy loot lists with the roster before production.

### Phase 06 - Complete Run Loop And Presentation

Finish story selection, ready/start, summary/reset, the supermarket presentation
and storage-room area, one scalable keyboard-operable UI, settings, fallback
assets, music/SFX channels, and three-run reset coverage. Run the first complete
story as a blind playtest and resolve any blocker in comprehension, party
downtime, run length, or willingness to replay before exit.
Complete departure/expiry transitions, empty-candidate leadership, pause/resume,
last-player continuation, the summary timeout, and the lifetime-window reset
on lobby entry and valid run start with injected-time tests, including start at
expiry and last-connection pause boundaries. Qualify failed settings replacement
on both supported OSes.

Store presence: once real screenshots exist, publish the Steam "Coming Soon"
page and the public site (`docs/site.md`) with wishlisting. Final store
materials are still approved in Phases 12-13. First publication must pass the
site's copy/media, legal/privacy, and working-contact checklist.

### Phase 07 - Multiplayer And Identity Hardening

Add Steam ticket validation and end-to-end Steam lobby creation, join, and
invite handling in isolated staging. Connect the owner's Steam membership flow
to the Phase 02 join grants, including owner handoff, and prove an authenticated
uninvited client cannot join directly using a known session ID. Qualify ownership
disagreement/failure and suppression of stale-membership re-grants after removal.

Add fresh-ticket reserved-seat reclaim, single-connection takeover, acknowledged
resync, replay protection, heartbeat, rate limits, slow-client handling, and
deterministic transport-chaos tests.
Qualify the Phase 06 lifecycle rules over real connections, including grace-deadline
rejoin, ballot discard, and content-mismatched takeover without reservation changes.
Freeze ticket service identity, ownership, cancellation/expiry/replay behavior
with real Steam evidence before production use. Inject provider timeout/outage,
rate refusal, and malformed responses to prove fail-closed admission without
damaging existing seats or connections.

Local and benchmark identity adapters remain available only to tests and
non-release builds and are proven absent from production packages.

### Phase 08 - Built-in content production

Complete the [release inventory and route minimum](mvp-contract.md#release-content-minimum).
Require inventory checks plus graph/transcript evidence for supported surviving
routes; duration remains a playtest gate. Integrate owner/friend/purchased assets
with written rights/provenance. Produce English text and fluent
collaborator-reviewed French text.

### Phase 09 - Playable MVP Stabilization

Run deterministic 2-, 3-, and 4-client scenarios, repeated run loops, reconnect
coverage, supported-resolution presentation checks, and risk-based regression
tests. Resolve blocker/critical defects before declaring Playable MVP.

### Phase 10 - Regional deployment and drain decision

Measure candidate Railway regions, creator-region session placement,
owning-process rejoin routing, graceful drain, forced stop, regional
loss, and rollback using the implemented game workload. Freeze acceptable
cross-area latency from controlled impairment playtests before choosing the
smallest region set.

Publish one decision record containing:

- the smallest Railway-only region set within the approved monthly hosting
  ceiling, measured usage/traffic/overlap costs, and creator-region placement;
- cross-region latency and uncertainty;
- owning-process session/rejoin routing;
- measured drain deadline and proof of the contract's admission/run-start
  refusal, idle-lobby closure, and summary-to-end behavior with connected peers;
- forced-stop and run-lost behavior;
- deployment and rollback procedure; and
- rejected alternatives with measured reasons.

The record must demonstrate the actual session placement and owning-process routing
mechanism through reconnect and deployment replacement. A generic regional
endpoint or readiness flag is not evidence of session affinity. Keep this a
measured feasibility gate before Phase 11 implementation.

### Phase 11 - Regional service and graceful drain

Implement the Phase 10 decision and locked drain transitions. Refuse new
admission, grants, and run starts; end idle lobbies and finish only current runs
through summary to `Ended`. Preserve eligible reserved-seat rejoin until session
end or the absolute deadline, then complete bounded cleanup without waiting for
clients to leave. Exercise regional routing, process replacement, forced stop,
crash/run-loss handling, observability, and rollback with production-shaped
integration tests. Distinguish maintenance closure from interrupted runs.

### Phase 12 - Release QA, Balance, And Performance

Provision production-equivalent staging. Run consented playtests, the release
QA matrix, calibrated client measurements, server capacity/load experiments,
regional drain/boundary workloads, and profiling of measured bottlenecks. Freeze release
budgets only after the reference environment and noise floor are recorded.
Freeze exact QA OS builds/UI scales and the `60 Hz` reference display mode;
frame gates measure the 60 FPS target cadence, not arbitrary monitor refresh.
Freeze the production/capacity artifact matrix and small authorized-account
workload from `docs/benchmark-plan.md`.

Prepare and assign owners for Steam store copy and media, age/content
disclosures, launch languages, pricing, privacy/support contacts, incident
handling, server operating cost, and shutdown policy.

### Phase 13 - Steam Packaging And Production

Create reproducible Linux/Windows packages and sign Windows artifacts. Verify
the Phase 07 Steam lobby/invite flow against final depots, deploy the production
server, drill rollback, and rerun release QA, client frame gates, and bounded server
performance/lifecycle checks against exact production digests. Rerun capacity
gates on the separately identified production-profile capacity build from the
same final revision. Require both evidence sets and the differences note
defined in `docs/benchmark-plan.md`. Approve and publish the store materials,
disclosures, end-user terms, privacy/support contacts, third-party notices,
asset provenance, final public site content, and operating plan.

## Post-MVP Boundary

Additional themes/UI, voice, durable active runs, normal solo play, and launch
locales beyond English and French belong to post-MVP plans.

## Definition Of Done

For every implementation phase:

- The acceptance behavior and failure oracle are clear before implementation.
- New behavior has focused public-interface tests for meaningful valid,
  boundary, and error paths.
- CI passes on a clean checkout for supported features and targets.
- Security-sensitive changes update the threat model and negative coverage.
- Performance-sensitive choices use the smallest production-mode measurement
  that can support the decision.
- Manual QA uses the shipped entrypoint only when a user journey exists.
- No blocker/critical defect or unexplained intermittent failure remains.
- The tracker links evidence; active implementation detail lives in issues.
