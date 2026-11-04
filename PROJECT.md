# Les Périssables ༼⁠ ⁠つ⁠ ⁠◕⁠‿⁠◕⁠ ⁠༽⁠つ

A stupid, funny, multiplayer tabletop-style RPG about supermarket food trying
not to die. Two to four friends pick premade characters, explore a grocery store,
make bad decisions, and survive dice checks and turn-based combat together.

**Status: planning.** There is no playable build yet. This document is the single
project plan: what we are making, how it works, and how we intend to release it.
The repository slug is `perissables`.

## Why this game?

The joke works because the rules are serious even when the characters are not.
A Banana Rogue and a dented Canned Beans Paladin should produce stories friends
retell after a disastrous roll.

Keep the good parts of tabletop co-op: shared choices, distinct characters,
uncertainty, and party chaos. Remove leveling, character builds, giant inventories,
and campaign commitments. A run should be easy to start, readable, and lethal
without wasting an entire evening.

## The game and its art direction

The setting, tone, and visual direction stay the same:

- Top-down 2D pixel-art exploration, with the readability of classic Pokémon or
  early Zelda. Use `16x16` tiles and four-direction `16x16` character frames.
- One supermarket map. Its storage room is an area of that map, with its own
  atmosphere, not a second theme or content pack.
- A separate combat view with the party and enemies facing each other.
- One readable, scalable UI: dialogue and choices, combat actions, HP bars, dice
  feedback, and four small inventory slots. No alternate skins.
- Visible keyboard focus, keyboard-only navigation, and readable text at
  `1280x720` and `1920x1080`. Target smooth `60 FPS` presentation.
- Music, ambience, and event-driven SFX. No voice playback or voice chat.
- English and French, with packaged fonts, proper accents, and text wrapping.

Example characters, not final balance sheets:

| Character | Personality and role |
| --- | --- |
| Banana Rogue | High Dexterity, slippery utility, cowardly flavor text |
| Canned Beans Paladin | Tanky, loud, righteous, probably dented |
| Leek Bard | Support spells, morale boosts, bad vegetable puns |
| Fish Sticks Mage | Risky burst damage and freezer-themed magic |

Example adventure: **Cleanup on Aisle 9**. The night crew vanished after opening
a pallet of cursed discount spices. Investigate a spill, break into storage,
calm a panicking NPC, and deal with possessed shopping carts, alarms, and a
Frozen Pizza Golem.

### One run

```text
Host / join through Steam
  -> lobby, character selection, ready
  -> supermarket exploration
  -> dialogue, group choices, dice checks
  -> turn-based encounters and loot
  -> mandatory boss, or an earlier party wipe
  -> summary
  -> cleared lobby for another run
```

Aim for a `35–45 minute` handcrafted route. Choices change local events, checks,
rewards, dialogue, and encounter details, but do not create separate campaigns
or bypass the boss. Replayability comes from party combinations and outcomes.
Measure duration in playtests; an estimate in a story file is not evidence.

### First release scope

| Content | Minimum |
| --- | --- |
| Story and world | One route; one supermarket map including storage |
| Playable characters | Four premade characters |
| Enemies | Five normal types and one mandatory boss |
| Spells and items | Eight character spells and eight items |
| Encounters per full surviving run | Two normal encounters and one boss |
| Story beats per full surviving run | Four to six checks, three major choices, `25–40` dialogue/choice beats |
| Music | Lobby, exploration, combat, and boss tracks |
| Other audio | Two ambience loops and at least twenty distinct SFX |

Early wipes naturally present less content. No builds, leveling, skill trees,
deep equipment, account progression, procedural maps, arbitrary scripts, public
matchmaking, or mid-run kicking. Starting a run requires two to four players;
an existing run may continue with one survivor. Solo starts are development-only.

Launch targets are Windows 11 x86_64 and Linux x86_64. Other platforms, Steam
Deck/controller support, achievements, additional themes, and further languages
are later decisions, not promises for this release.

## Gameplay rules

The technology changes, not the game's rules. The host resolves these rules for
everyone; clients never decide their own successful actions.

### Party-size balance

- Two, three, and four players are all supported launch states.
- Four players is the full-party balance target. Two- and three-player runs must
  remain playable and comparably challenging, not just a harder or easier copy
  of the four-player fight.
- Decide and document how encounters scale before authoring content. Use
  party-size encounter variants or explicit content-defined enemy count/HP
  changes; do not ship fixed encounters and hope the dice compensate.
- Balance the two normal encounters and the boss at all three party sizes.
  Consistent triviality, brutality, or long waiting turns at any size is a
  release blocker.

### Dice

- Six stats: Strength, Perception, Chance, Dexterity, Charisma, and Education.
  Base stats are integers in `5..=70`.
- Roll uniformly from `0..=100`, including both endpoints. `0`, displayed as
  `000`, is critical success; `100` is critical failure. Otherwise a roll at or
  below the check target succeeds.
- For an unmodified stat, normal success probability is `stat / 101`; including
  the critical it is `(stat + 1) / 101`.
- The host owns the run's random generator. Rejected actions consume no random
  draws. Tests supply known random states and explicit time.

### Exploration and story

- Continuous cardinal movement, no diagonals. The host decides speed, collision,
  and final positions. Interact with a valid object directly ahead within one tile.
- Any connected living player can initiate an interaction. Only one party-wide
  sequence runs at a time, including its story, combat, and loot. Freeze world
  positions and clear held movement until exploration resumes; do not queue
  extra interactions or movement for later.
- Stories are data-driven state machines with dialogue, check, encounter,
  transition, return, and end nodes. Effects use supported flag/item operations,
  never story-specific executable code. Bound automatic transition chains.
- Dialogue advances when all connected living players press continue, or after
  `20 seconds` of unpaused gameplay. Disconnect discards a player's continue press.
- Pick a run leader uniformly at run start. Story checks use the connected living
  leader, otherwise the connected living player with the lowest lexical player ID.
  Selecting the actor, applying modifiers, rolling, and committing happen together.
  With no eligible actor, wait without spending anything; ending the run cancels it.
- Checks require success/failure branches. Missing critical branches use the
  corresponding normal branch while keeping critical feedback.
- Each story vote has a fixed set of choices and a visible timeout default.
  Connected living players have one replaceable ballot. Close after `45 seconds`
  of unpaused gameplay, or once everyone eligible has voted. A ballot at or after
  the deadline is too late.
- The choice with the most votes wins. A tied leader ballot wins if the leader is
  connected and living; otherwise use the tied choice backed by the lowest lexical
  player ID. With no ballots, select the declared default. Voting consumes no randomness.
- Disconnect, death, or departure discards ballots. Reconnect does not restore
  them. A disconnected leader keeps the role during their seat reservation but
  cannot break ties. A dead connected leader may transfer it to a connected living
  player. Departure/expiry vacates the role: choose a connected living replacement
  uniformly, or wait without a random draw until someone eligible reconnects.

### Combat, effects, and inventory

- Fixed positive content-defined HP, not HP derived from stats. Each run starts
  at maximum HP, empty inventory, and no effects. HP, death, and items persist
  between encounters; no passive healing or resurrection. Spell charges reset
  each encounter. A new run resets all run-owned state.
- Turn order is descending Dexterity, then lexical actor ID. One attack, spell,
  item, or pass per turn. There is no combat grid or range. A player has
  `30 seconds` of unpaused gameplay to act, then passes automatically.
- Attack checks Strength. Success deals `max(1, floor(Strength / 5))` damage;
  critical success doubles it; failures deal zero.
- A spell names its check stat, effect, target type, magnitude where applicable,
  and per-encounter charges. Casting spends one charge and the turn, even on failure.
- Five effects: direct damage, healing, guard, next-check bonus, next-check penalty.
  Each spell/item applies one effect to one target. Healing caps at maximum HP.
- Successful spells use their authored magnitude. Critical success doubles damage,
  healing, or modifier magnitude; critical guard protects two hits instead of one.
  Normal failure has no effect. Critical failure redirects damage/penalties to a
  uniformly random living member of the caster's side, including the caster, and
  healing/guard/bonuses to a uniformly random living opponent. No target means no effect.
- Guard halves incoming positive damage, rounded down with a minimum of one,
  and consumes one protected hit. Each actor holds at most one guard, one bonus,
  and one penalty; reapplication replaces the same kind even when weaker.
  Zero damage does not consume guard.
- The next attack, spell, or story check consumes both modifiers and checks
  `stat + bonus - penalty`, clamped to `0..=100`. Critical extremes still win.
  Passes, item uses, and rejected actions do not consume check modifiers. Effects
  clear on death, removal, encounter end, or run reset, not on disconnect.
- Enemies choose a legal action uniformly from their kit, then choose a valid target
  uniformly. With no legal action they pass. Kits may contain only spells. No separate AI system.
- Four inventory slots, one item per slot, filled lowest-index first. Items work
  only on the holder's combat turn, without a roll, and spend the item and turn
  together. Damage/penalties target living opponents; healing/guard/bonuses target
  living allies including self, or self only, as authored. A legal full-HP or
  replacement use still spends the item. Invalid actions spend nothing.
- Story grants resolve in authored order. Skip a recipient without a living
  reserved character or free slot, with feedback. No replacement or reward queue.
- Victory offers each defeated enemy's authored zero-to-two-item loot list, not
  random drops or its held inventory. Player corpses and wipes create no loot.
- Each loot item gets replaceable ballots for a living reserved recipient with a
  free slot. All items share one `45 second` phase, closing early only when every
  eligible voter has voted on every item. Resolve in lexical corpse ID then loot-slot
  order, rechecking eligibility and space before each award. Use the story tie
  rule, but no default recipient: unassigned items disappear. Run end cancels loot.
- Dead players spectate, cannot vote or act, and cannot be living targets. Their
  only gameplay control is leader transfer when applicable. Enemy defeat wins;
  party wipe or a bounded engine round limit loses, including in the boss fight.

## Technical approach

| Area | Choice |
| --- | --- |
| Language | Odin, with a pinned and tested compiler release |
| Window, rendering, input, audio | raylib through Odin's bundled bindings |
| Steam integration | Steamworks SDK through verified bindings or a small native adapter |
| Lobby and invites | Steam lobbies, private or friends-only |
| Gameplay transport | Steam Networking Sockets with Steam Datagram Relay |
| Simulation | Authoritative rules running in the host player's process |
| Stories and definitions | Repository-owned JSON |
| Map authoring | Tiled, packaged TMX with collision/event objects |
| Local preferences | Settings file; no authoritative save data |
| Distribution | Windows/Linux builds uploaded through SteamPipe |

Odin suits a small game with explicit data and direct control flow. Its vendor
collection includes raylib bindings. Memory management is manual: allocations,
native handles, callbacks, and cleanup need clear ownership. raylib is a library,
not an editor; UI layout, text wrapping, and game tools are still our responsibility.

Keep one game executable and ordinary modules for rules, presentation, networking,
and content. Rules must be testable without a window or Steam connection. The host
player's own actions go through the same validation as remote actions. No custom
engine framework, plugin system, or separate web backend.

Pin the compiler, raylib, Steamworks SDK, and binding revisions when the first
Windows/Linux integration works. Record build instructions here then; there are
no build commands to run today. Verify licensing and redistribution requirements
for every packaged dependency.

## Online play and servers

### Who runs the server?

One player clicks **Host**. Their computer runs the authoritative simulation and
their own client presentation. Other players click an invite or join through Steam.

```text
                  Steam lobby and invites
                            |
                  Host player's game
                  rules + local presentation
                      /             \
             Steam networking / relay
                    /                 \
             Guest player         Guest player
```

We do not rent or operate gameplay servers for this release. No regional fleet,
database, custom accounts, publisher-key web service, or deployment-drain system.
Steam supplies discovery and connection/relay services, not the game simulation.
Lobbies do not carry gameplay messages; the networking connection does that.

This removes server hosting bills and regional operations, not networking work or
dependence on Steam's availability. It also changes the trust model: a modified
host can cheat or inspect state. We accept that for private co-op without rankings,
an economy, or persistent progression. Dedicated servers are not a launch promise.

### Joining and synchronization

- The host authorizes peers using their authenticated Steam identity and current
  lobby membership. A claimed player ID or known lobby ID is not permission.
  Handle removals so stale membership callbacks cannot immediately readmit a peer.
- Only the host selects the story, removes seats in the lobby, and starts runs.
  Start requires two to four connected, ready players with unique characters.
  Membership, story, or character changes clear readiness. No mid-run kicking.
- New seats join only before a run. Require matching protocol, rules, and built-in
  content versions/checksum before admission or reconnect takeover. Show clear
  update guidance on mismatch; never download content from lobby metadata.
- Guests send intentions, not outcomes. The host validates phase, identity,
  sequence, target, and bounds before mutation, then sends results and snapshots.
  Use ordered action IDs/revisions to reject replay and avoid duplicate effects.
- Send recipient-specific state, not random-generator state or other players'
  private inventories. Static story files shipped to players are inspectable.
- Keep messages, queues, per-peer work, and retries bounded. Reject malformed
  traffic, rate-limit input, and disconnect slow peers without stalling the party.
- Target a `20 Hz` simulation/network update with coalesced movement/state and
  `60 FPS` presentation. Predict local movement, reconcile to host positions,
  and interpolate remote players. Prediction never triggers authoritative events.
- Poll Steam callbacks and network events without blocking rendering. Start with
  a simple owned game loop rather than a background task framework.

### Disconnects, run end, and updates

- A disconnected guest keeps their seat for an initial `10 minute` grace period.
  Reconnect binds the same Steam identity, restores the reserved character through
  a full state resync, and acknowledges it before accepting new actions. Allow
  only one active connection per seat; stale disconnects cannot evict its replacement.
- While others play, a disconnected character stays targetable and its turns time
  out to pass. Explicit leave or grace expiry removes the character, inventory,
  effects, and ballots, without corpse loot. It cannot rejoin that ongoing run.
- With no connected living players, pause gameplay and gameplay timers, even if
  the host is watching as a dead spectator. Seat expiry stays absolute. Apply due
  expiries before new input or deadline resolution; pause/resume cannot reroll events.
- One living reserved character may finish an existing run, including the host
  alone after guests leave. No living reserved characters means a wipe; no occupied
  seats means the session ends. Host loss ends the session independently, as below.
- Summary returns remaining seats to a cleared lobby after all acknowledge or an
  absolute `2 minute` timeout. Reset run state, not the players' local preferences.
- If the host leaves, crashes, or becomes unreachable beyond the connection
  timeout, the session ends. Show connection status before declaring loss, then
  explain that the run was lost. No host migration or crash recovery at launch.
  Steam may transfer lobby ownership automatically; that does not transfer game state
  or authorize a replacement host for the old run.
- Updates ship through Steam. An already-running compatible party can keep playing;
  an updated guest may need to wait for the host to restart before joining. Test
  and explain this rather than trying to hot-update or migrate active runs.

## Content and production

Keep stories, characters, enemies, spells, items, maps, localized text, and assets
in this repository. Engine code defines the five effects and all resolution rules;
JSON assembles them into the supermarket adventure. No executable content scripts.

Keep the finite orthogonal TMX map with `16x16` tiles and explicit collision/event
objects. Embed tileset definitions; presentation references resolve only to packaged
assets. Parsing must not fetch external files or network resources.

Validate content before play: known fields, unique IDs, valid references, bounded
values/resources, reachable branches, and terminating automatic transitions.
Partial development content is allowed; release checks separately verify the full
inventory, per-route counts, and mandatory boss. Playtests establish run length.

Preserve editable art/audio sources and export steps. Use Ogg Vorbis for music and
ambience, signed 16-bit PCM WAV for SFX, and mono/stereo `48 kHz` audio. Package
fonts for English/French; have a fluent collaborator review French text. Missing
presentation assets get safe fallbacks, but missing required game data prevents play.

Every asset needs a recorded creator, source, license/permission, and credit.
Friends' contributions need written rights covering modification and distribution,
including Steam. No placeholders, unlicensed assets, or developer-only actions in
the release package.

Settings cover display, separate audio volumes, keybindings, and accessibility.
Use standard per-user data locations on each OS. Save through a temporary file and
safe replacement; report failures without destroying the previous usable settings.
Preserve malformed/future files and use in-memory defaults instead of crashing.

## Testing and release quality

Keep verification proportional to a four-player game:

- Automated rule tests for dice extremes, voting, effects, inventory, combat,
  departures, deadlines, and reset. Inject time/randomness; do not use sleeps or
  statistical assertions where exact outcomes can be tested.
- Content checks for invalid data, missing references/assets, supported routes,
  and English/French text. Do not duplicate gameplay logic in test expectations.
- Two-, three-, and four-player tests for convergence, rejected/duplicate input,
  guest reconnect, host loss, slow peers, version mismatch, and three consecutive
  lobby -> run -> summary -> lobby loops.
- Clean-install tests through the actual Steam packages on Windows and Linux,
  including cross-platform invites, keyboard navigation, scaling, audio, settings,
  and disconnect feedback. Name the supported Linux baseline once tested.
- Measure frame pacing and responsiveness on named machines and realistic impaired
  connections. Profile actual problems rather than speculative scaling workloads.
- Playtest with people outside implementation. Record confusion, waiting time,
  abandoned runs, duration, and whether they want to play again. Test fun before polish.

Build and run focused tests in CI on both targets as implementation arrives. Keep
the first failure visible; do not retry until green. Review dependency advisories
and native memory/resource cleanup. Test only local or explicitly authorized peers;
never load-test Steam. Do not ship credentials, test identity bypasses, or logs
containing tickets, private messages, or unnecessary personal data. No optional
telemetry or automatic crash uploads by default.

Ship only when the full content is present, required journeys work on both packaged
targets, performance is acceptable, and no known blocker remains. Record any accepted
non-blocking issues. A plan, successful compile, or one happy-path run is not release QA.

## Publishing and maintaining the Steam release

1. Complete Steamworks onboarding, identity/bank/tax information, and the
   **$100 USD Steam Direct fee per product**. Check Valve's current waiting periods
   and review lead times before choosing a release date.
2. Obtain the app ID and SDK access. Prove raylib, Steam initialization, lobby
   invites, and networking on Windows and Linux early, using authorized test accounts.
   Verify or implement only the Steam bindings needed; do not assume they are bundled
   with Odin. Never put a Steam publisher Web API key in the game.
3. Prepare the store page: honest description, capsule art, real screenshots,
   trailer, languages, system requirements, pricing, content/age disclosures,
   support/privacy contacts, and accurate online requirements. Clearly disclose
   that the host must remain connected and active runs are not saved.
4. Publish the Coming Soon page and meet its required public visibility period.
   Show actual gameplay, collect wishlists, and playtest before promising a date.
5. Upload separate Windows/Linux depots through SteamPipe. Include required runtime
   libraries and third-party notices, and keep SDK tools and secrets out of packages.
   Use a private beta branch to test clean installs, updates, and mixed-OS parties.
6. Complete Valve's store/build reviews and all release checklists. Run final QA on
   the exact candidate packages, then release when ready, not just when approved.
7. Keep a known-good build for rollback. Test patches on the beta branch, publish
   release notes, monitor player reports, and maintain a working support contact.
   Keep version mismatch and host-loss guidance understandable after updates.

An optional static website can present the game, screenshots/trailer, changelog,
news, Steam link, and legal/privacy/support information. It is not a backend and
never participates in gameplay. Publish only approved media and a privacy notice
matching the actual host, logs, and embeds; no analytics by default.

## Milestones and current progress

No calendar estimate until the first online slice works. Track active tasks in
issues; keep only milestone completion and important decisions here.

- [x] Agree on the premise, art direction, Odin + raylib, and player-hosted Steam model.
- [ ] **Prove the stack.** Pin tools, compile both targets, and connect two machines
  through Steam. Prove invite authorization, cross-platform play, guest reconnect,
  and host-loss feedback before building the full game around the integration.
- [ ] **Make one tiny online run.** Two players select characters, move, trigger one
  interaction, roll one check, resolve combat, and reach the same summary. Put two
  people through it immediately, with placeholder graphics if needed.
- [ ] **Complete the loop.** Implement the remaining rules, content validation,
  supermarket map, scalable UI, settings, localization, and audio. Test repeated
  runs and failure paths as they arrive.
- [ ] **Produce and playtest the adventure.** Finish the release inventory and
  rights/credits. Fix pacing, comprehension, balance, and willingness to replay.
  Prepare the store page once real screenshots exist.
- [ ] **Release on Steam.** Qualify both depots, finish store/legal/support work,
  pass reviews and final QA, and keep a tested rollback build.

## Reference links

- [Odin documentation](https://odin-lang.org/docs/) and
  [bundled raylib bindings](https://github.com/odin-lang/Odin/tree/master/vendor/raylib)
- [raylib](https://www.raylib.com/)
- [Steam lobbies](https://partner.steamgames.com/doc/features/multiplayer/matchmaking)
- [Steam Networking Sockets](https://partner.steamgames.com/doc/api/ISteamNetworkingSockets)
  and [Steam Datagram Relay](https://partner.steamgames.com/doc/features/multiplayer/steamdatagramrelay)
- [Steam Direct](https://partner.steamgames.com/steamdirect) and
  [uploading builds through SteamPipe](https://partner.steamgames.com/doc/sdk/uploading)

## License

This repository is proprietary, All Rights Reserved. See [LICENSE](LICENSE) and
[COPYRIGHT](COPYRIGHT). No permission is granted to copy, modify, redistribute,
or reuse the code or bundled assets without prior written permission from the owner.
Outside pull requests are not accepted until a reviewed contribution policy and
appropriate written contributor agreement are in place. Issues and non-code
feedback remain welcome.
