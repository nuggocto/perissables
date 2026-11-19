# Les Périssables ༼⁠ ⁠つ⁠ ⁠◕⁠‿⁠◕⁠ ⁠༽⁠つ

A stupid, funny, multiplayer tabletop-style RPG about supermarket food trying
not to die. Two to four friends pick premade characters, explore a grocery store,
make bad decisions, and survive dice checks and turn-based combat together.

**Status: planning the digital adaptation.** We already play this game with
friends at the table. There is no playable digital build yet. This document is the single
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

Setting, tone, and visual direction:

- Top-down 2D pixel-art exploration, with the readability of classic Pokémon or
  early Zelda. Use `16x16` tiles and four-direction `16x16` character frames.
- The world renders at a virtual `320x180` (20 by about 11 tiles visible) and the
  UI at `640x360`, both scaled by whole numbers only: world `x4` and UI `x2` at
  `1280x720`, world `x6` and UI `x3` at `1920x1080`, and so on. Non-16:9 screens
  get black bars, never stretching or fractional scaling.
- One supermarket map. Its storage room is an area of that map, with its own
  atmosphere, not a second theme or content pack.
- A separate combat view with the party and enemies facing each other.
- One readable, scalable UI: dialogue and choices, combat actions, HP bars, dice
  feedback, four small inventory slots, and a rules page in the pause menu. No
  alternate skins.
- Visible keyboard focus, keyboard-only navigation, and readable text at
  `1280x720` and `1920x1080`. Target smooth `60 FPS` presentation.
- Music, ambience, and event-driven SFX. No voice playback or voice chat.
- English and French are the first language candidates. Confirm each language
  after a writer and native reviewer can play the finished scenes (see **Localization**).

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
| Onboarding | The route's first check and first encounter teach the rules in dialogue |
| Music | Lobby, exploration, combat, and boss tracks |
| Other audio | Two ambience loops; distinguish a hit, miss, guard and critical by ear in a fight |

Early wipes naturally present less content. No builds, leveling, skill trees,
deep equipment, account progression, procedural maps, arbitrary scripts, public
matchmaking, or mid-run kicking. Starting a run requires two to four players;
an existing run may continue with one survivor. Solo starts are development-only.

Launch targets are Windows 11 x86_64 and Linux x86_64. Other platforms, Steam
Deck/controller support, Remote Play Together (it needs controllers to tell
players apart), a demo, achievements, additional themes, Traditional Chinese,
right-to-left languages such as Arabic, and further languages
are later decisions, not promises for this release.

## Gameplay rules

The host resolves these rules for everyone; clients never decide their own
successful actions.

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
- Roll uniformly from `1..=100`. `1` is critical success; `100` is critical
  failure. Otherwise a roll at or below the check target succeeds.
- A stat is its exact success chance: stat `45` succeeds on `45%` of rolls,
  criticals included (`1%` critical, `44%` normal). The UI shows the actor, the
  stat, and the final chance before every roll.
- After modifiers, show the actual success chance: `clamp(target, 1, 99)%`.
  Even at target `0`, a `1` succeeds; even at target `100`, a `100` fails.
- The host owns the run's random generator. Rejected actions consume no random
  draws. Tests supply known random states and explicit time.

### Exploration and story

- Tile-stepped cardinal movement, one `16x16` tile per step, no diagonals. A held
  direction repeats steps at a fixed host-defined rate. The host accepts or
  rejects each step and owns final positions. Players do not block each other.
  Interact with a valid object on the tile directly ahead.
- Any connected living player can initiate an interaction. Only one party-wide
  sequence runs at a time, including its story, combat, and loot. Freeze world
  positions and clear held movement until exploration resumes; do not queue
  extra interactions or movement for later.
- Before freezing the scene, identify the initiator and object to everyone
  ("Leek Bard rang the freezer bell"). In exploration, try one keyboard ping
  that marks the caller's tile and character for the party. It starts no event.
- Stories are data-driven state machines with dialogue, check, encounter,
  transition, return, and end nodes. Effects use supported flag/item operations,
  never story-specific executable code. Bound automatic transition chains.
- "Connected players" includes dead ones. For the prototype, dialogue waits
  for each connected reader to continue. Show who is still reading. Disconnect
  removes that reader from the wait. Test a pair with different reading speeds
  before setting any automatic advance timer.
- A story vote can choose an approach before a check. Then the initiator names
  one connected living volunteer (self is valid); show each candidate's stat
  and odds. No second vote. For online play, if nobody is nominated within
  `20 seconds` of unpaused gameplay, the initiator rolls. If that roller or a
  nominated roller is ineligible, use the lowest connected living seat. If the initiator
  leaves before nominating, use that fallback immediately. Recheck eligibility
  and resolve the check in one step. With no eligible actor, wait without spending anything.
- Checks require success/failure branches. Missing critical branches use the
  corresponding normal branch while keeping critical feedback.
- Each story vote has a fixed set of choices and a visible timeout default.
  Each connected player, living or dead, has one replaceable ballot. Close after
  `45 seconds` of unpaused gameplay, or once every connected player has voted. A
  ballot at or after the deadline is too late.
- The choice with the most votes wins. On a tie, the host rolls one die with a
  side for each tied choice, uniformly, and every player sees the roll before
  the result applies. With no ballots, select the declared default. A tie roll
  is the only random draw voting makes.
- Disconnect or departure discards ballots. Reconnect does not restore them.

### Combat, effects, and inventory

- Fixed positive content-defined HP, not HP derived from stats. Each run starts
  at maximum HP, empty inventory, and no effects. HP, death, and items persist
  between encounters; no passive healing, and the revive item is the only way
  back from death. Spell charges reset each encounter. A new run resets all
  run-owned state.
- Turn order is descending Dexterity, then players by seat index, then enemies in
  the encounter's authored order. One attack, spell, item, or pass per turn.
  In the prototype, the Paladin can instead **Cover** a living teammate:
  apply ordinary one-hit guard without rolling, spending the Paladin's turn.
  There is no combat grid or range. A player has `30 seconds` of unpaused
  gameplay to act, then passes automatically.
- Each character and enemy has one authored basic attack naming its check stat
  and a fixed positive damage. Success deals that damage; critical success
  doubles it; failures deal zero. Author each attack on a stat the actor is good
  at, so a low-Strength character still has a turn worth taking. Damage does not
  derive from the stat.
- A spell names its check stat, effect, target type, magnitude where applicable,
  and per-encounter charges. Casting spends one charge and the turn, even on failure.
- Six effects: direct damage, healing, guard, next-check bonus, next-check penalty,
  and revive. Each spell/item applies one effect to one target. Healing caps at
  maximum HP.
- Revive is item-only. Keep revive stickers on the party's shared sheet so the
  only sticker cannot get stuck in an Expired character's pocket. A connected
  living player can spend one on a dead reserved ally as their combat action,
  or during free exploration. Restore half maximum HP rounded down (minimum
  one), clear effects, and keep spent charges spent. In combat the ally acts
  next round; in exploration they can act immediately. The full route offers
  at most two stickers. A wipe still ends the run.
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
- Each kit entry has an optional positive integer weight (default `1`). Enemies
  choose a legal action with probability proportional to its weight, then choose
  a valid target uniformly. With no legal action they pass. Kits may contain only
  spells. Weights are data, not a separate AI system.
- Four personal inventory slots, one ordinary item per slot, filled lowest-index
  first. Revive stickers use no personal slot. Inventories
  are visible to the whole party. Ordinary items work
  only on the holder's combat turn, without a roll, and spend the item and turn
  together. Damage/penalties target living opponents; healing/guard/bonuses target
  living allies including self, or self only, as authored. A legal full-HP or
  replacement use still spends the item. Invalid actions spend nothing.
- During free exploration, a living player may discard one of their
  own items, permanently. There is no trading between players.
- Personal item grants resolve in authored order. Skip a recipient without a living
  reserved character or free slot, with feedback. No replacement or reward queue.
- Author rewards by scene. When there is a tradeoff, present one ordinary story
  vote over the whole offer (for example, claim a revive or spend the coupon to
  slow the cart). State who gets what before voting; resolve the winning branch
  with the story grant rules. No per-item ballots or automatic corpse loot.
- Dead characters are shown as "Expired". Their players keep voting on story
  choices (including rewards) and pressing continue, but cannot move, act in combat, roll
  checks, or start interactions. Only revive can target them.
- Enemy defeat wins. Party wipe loses. So does the end of round `20` ("the store
  is closing"), including in the boss fight. Show rounds left from the first
  turn; call out the last five. Test whether anyone changes a decision because
  of closing time before tuning the limit.

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
| Map authoring | Tiled, packaged JSON map (`.tmj`) with collision/event objects |
| Text | `kb_text_shape` + `stb_truetype` from Odin's vendor collection, own glyph atlas |
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

Code shape, decided before the first line:

- Host run state is one plain struct with fixed-capacity arrays and no pointers.
  Seats, enemies, items, and story nodes refer to each other by index. Copy the
  struct for test fixtures and derive a separate public snapshot for clients.
- Content IDs are strings only in JSON. Loading resolves them to `distinct`
  integer IDs, and the rules never compare strings.
- Three lifetimes: a content arena loaded once, a run arena freed at run reset,
  and the temp allocator reset every frame.
- `core:encoding/json` unmarshal skips fields it does not know. The "known
  fields" content check needs its own pass over the parsed value.
- Odin bundles no Steamworks binding. Bind the flat C API for the calls we use.
  Dispatch Steam callbacks manually from the main loop, and set `context` at the
  top of every `"c"` callback before it allocates or logs.
- Encode network messages field by field with bounds checks. Do not send raw
  struct memory.
- Do not use raylib's text functions: they draw one glyph per codepoint, which
  breaks Thai marks and cannot follow Chinese, Japanese, or Thai line-break
  rules. Text goes through `vendor:kb_text_shape` (segmentation, line breaks,
  shaping), `vendor:stb/truetype` (rasterizing by glyph index), and our own glyph
  atlas texture drawn with raylib. Both libraries ship with Odin. The hot-seat
  build may use raylib's text functions with placeholder English text; the
  shaped text path is part of **Complete the loop**.
- The Steamworks SDK is never committed; its terms do not allow republishing it.
  Download it from the Steamworks partner site into `vendor/steamworks/`, which
  is gitignored, as is the development `steam_appid.txt`.

Start on the current compiler release (`dev-2026-11a` when this was written). Pin
the compiler, raylib, Steamworks SDK, and binding revisions when the first
Windows/Linux integration works. Record build instructions here then; there are
no build commands to run today. Verify licensing and redistribution requirements
for every packaged dependency.

Windows builds need a Windows machine or CI runner. Build Linux release binaries
inside the Steam Linux Runtime SDK container, not on the development machine, so
the binary does not require a newer glibc than players have. Confirm this when
naming the supported Linux baseline.

## Online play and servers

The networking and disconnect rules in this section are a draft. Revisit them
once the **Prove the stack** milestone works; the hot-seat build does not depend
on them.

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
  Use ordered action IDs to reject replays and duplicates. The host assigns an
   input-window ID to each dialogue beat, story vote, roller nomination, combat turn,
  and entry into exploration. These IDs are not reused within a session,
  including across runs. The public snapshot carries the current ID. Clients
  attach the ID they see when they capture the input. Reject any gameplay
  action that does not match the current window before it changes gameplay
  state, even if its action ID is new.
- Every guest gets the same full public snapshot, inventories included. Send it
  whole on each change and use the same message for reconnect resync. The host
  keeps random-generator state out of this snapshot and all other messages.
  Static story files shipped to players are inspectable.
- Keep messages, queues, per-peer work, and retries bounded. Reject malformed
  traffic, rate-limit input, and disconnect slow peers without stalling the party.
- Target a `20 Hz` simulation/network update and `60 FPS` presentation. A guest
  sends one step intent per tile. Its client starts the step animation at once.
  The host checks the step against the current exploration phase, authoritative
  position, movement rate, and collision map. If the host rejects the step, the
  client corrects to the host's reported position. Other players animate between
  the tiles the host reports. There is no continuous prediction or
  reconciliation, and a local step never triggers an event.
- Poll Steam callbacks and network events without blocking rendering. Start with
  a simple owned game loop rather than a background task framework.

### Disconnects, run end, and updates

- A disconnected guest keeps their seat for an initial `10 minute` grace period.
  Reconnect binds the same Steam identity, restores the reserved character through
  a full state resync, and acknowledges it before accepting new actions. Allow
  only one active connection per seat; stale disconnects cannot evict its replacement.
- While others play, a disconnected character stays targetable and its turns
  pass immediately, without waiting for the turn timer. Explicit leave or grace
  expiry removes the character, inventory, effects, and ballots, without corpse
  loot. It cannot rejoin that ongoing run.
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
in this repository. Engine code defines the six effects and all resolution rules;
JSON assembles them into the supermarket adventure. No executable content scripts.

Keep the finite orthogonal Tiled map with `16x16` tiles and explicit collision/event
objects, exported as JSON (`.tmj`) so maps and definitions share one parser. Store
layer data as plain uncompressed arrays. Embed tileset definitions; presentation
references resolve only to packaged assets. Parsing must not fetch external files
or network resources.

Validate content before play: known fields, unique IDs, valid references, bounded
values/resources, reachable branches, and terminating automatic transitions.
Partial development content is allowed; release checks separately verify the full
inventory, per-route counts, and mandatory boss. Playtests establish run length.

Preserve editable art/audio sources and export steps. Audio files and large
binary sources (`.wav`, `.ogg`, `.flac`, `.psd`, `.kra`) go through Git LFS, as
set in `.gitattributes`; run `git lfs install` once per clone. PNG, Aseprite,
Tiled, and JSON files are small and stay in normal Git. Use Ogg Vorbis for music and
ambience, signed 16-bit PCM WAV for SFX, and mono/stereo `48 kHz` audio. Missing
presentation assets get safe fallbacks, but missing required game data prevents play.

Every asset needs a recorded creator, source, license/permission, and credit.
Friends' contributions need written rights covering modification and distribution,
including Steam. No placeholders, unlicensed assets, or developer-only actions in
the release package.

Settings cover display, separate audio volumes, keybindings, and accessibility.
Use standard per-user data locations on each OS. Save through a temporary file and
safe replacement; report failures without destroying the previous usable settings.
Preserve malformed/future files and use in-memory defaults instead of crashing.

### Localization

Start with the language the scene writer knows best; English and French are the
first candidates for release. Simplified Chinese, Japanese, Korean, Thai,
Polish, Czech, Turkish, and Russian are candidates for later, with no promised
date. Ship a language only when someone owns the adaptation, every string is
translated, and a native speaker has reviewed the jokes and layout in the game.
Only then list it on Steam.

Candidate fonts (SIL OFL); package and check the variants needed by shipped languages:

| Languages | Font |
| --- | --- |
| English, French, Polish, Czech, Turkish, Russian | Fusion Pixel 12px proportional, `latin` variant |
| Simplified Chinese, Japanese, Korean | Fusion Pixel 12px proportional, `zh-Hans`, `ja`, and `ko` variants |
| Thai | Noto Sans Thai (vector; Thai marks need more than 12 pixels) |

- Use the matching regional variant: the same Han character is drawn differently
  in Chinese and Japanese, and native readers notice the wrong one. Confirm each
  font covers every character its language uses; the content check enforces it.
- Text is drawn at native resolution, laid out in the `640x360` UI coordinates.
  Pixel fonts render at whole multiples of `12px` with hard edges, so they match
  the UI's pixels; the Thai font renders smoothly at the same scale. Each
  language sets its own line height.
- Line breaks: spaces for Latin, Cyrillic, and Korean; between characters for
  Chinese and Japanese, without starting a line on closing punctuation such as
  `。、」` or ending one on opening punctuation. Thai has no spaces between words,
  so translators put a zero-width space (`U+200B`) at word boundaries. The
  content check flags long Thai runs without one.
- Give dialogue, choices, and UI text string IDs from the first digital build.
  Localized strings live in one JSON file per language. Use
  named placeholders (`{actor}`, `{item}`), never concatenated fragments, so each
  language can reorder words. Plurals use CLDR categories (Russian, Polish, and
  Czech need several forms; Chinese, Japanese, Korean, and Thai need none).
- Never change case in code. Turkish uppercases `i` to `İ`, so store text exactly
  as displayed.
- Network messages carry IDs and numbers, never display text. Each client shows
  its own language, so mixed-language parties work.
- The default language follows Steam's game language and can be changed in
  settings at any time, which rebuilds the glyph atlas.
- Steam names can contain any script. Shape them on lobby join through every
  packaged font in turn, and draw a placeholder box for anything none of them
  cover, such as emoji. During the run, players are shown by character, not by
  Steam name.
- The first digital build includes a pseudo-language that lengthens every string by 40% and
  mixes in accented, Cyrillic, and CJK characters, so layout overflow and missing
  glyphs show up before real translations exist.
- Translate after the source text freezes, not during writing. Puns
  and jokes need adapting, not literal translation; translators get context
  (speaker, scene, character limits) for each string. Every translator is
  credited and needs written rights like any other contributor.

## Testing and release quality

Keep verification proportional to a four-player game:

- Automated rule tests for dice extremes, voting, effects, inventory, combat,
  departures, deadlines, and reset. Inject time/randomness; do not use sleeps or
  statistical assertions where exact outcomes can be tested.
- Content checks for invalid data, missing references/assets, supported routes,
  and each shipped language's text, fonts, and placeholders. Do not duplicate gameplay
  logic in test expectations.
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
   Run outside playtests through Steam Playtest rather than handing out builds.
   Steam Next Fest needs a demo, so it waits on the later demo decision.
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

## Team and roles

| Role | Owner |
| --- | --- |
| Design, code, builds, Steam release | Nugget |
| Pixel art (characters, enemies, tileset, combat view, UI) | Unassigned |
| Music and SFX | Unassigned |
| Translation and native review (one per language) | Unassigned |

Name an owner, or a budget for commissioned or licensed work, for every role
before **Produce and playtest the adventure** starts. Placeholder art is fine
until then.

## Milestones and current progress

No calendar estimate until the first online slice works. Track active tasks in
issues; keep only milestone completion and important decisions here.

- [x] Agree on the premise, art direction, Odin + raylib, and player-hosted Steam model.
- [ ] **Bring one familiar scene to the screen.** Pick a scene from an actual
  session with friends and implement its choices, rolls, and consequences in a
  local hot-seat build with placeholder graphics. Use the table rules as the
  reference. Check that each player can tell whose decision it is, what they
  can do, and why a roll changed the outcome.
- [ ] **Prove the stack.** Pin tools, compile both targets, and connect two machines
  through Steam. Prove invite authorization, cross-platform play, guest reconnect,
  and host-loss feedback before building the full game around the integration.
- [ ] **Make one tiny online run.** Two players select characters, move, trigger one
  interaction, roll one check, resolve combat, and reach the same summary. Put two
  people through it immediately, with placeholder graphics if needed.
- [ ] **Complete the loop.** Implement the playtested rules, content validation,
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
