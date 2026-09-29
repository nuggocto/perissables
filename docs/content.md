# Built-in content

Authority: game-facing content requirements come from
`docs/mvp-contract.md`. Exact schema fields, limits, canonical bytes, and
positive/rejection fixtures freeze with the versioned built-in schema.

## Purpose

Stories, preset characters, enemies, items, and presentation references are data
so the one release story can be edited without moving gameplay authority into
content. Engine-owned rules remain fixed.

One session uses the immutable aggregate identity defined in
[the content contract](mvp-contract.md#content-contract). Phase 02 supplies
admission fixtures and Phase 04 supplies canonical checksum fixtures. Client and
server share the schema; an identity match does not establish client trust.

## Data and engine boundary

Content controls:

- story nodes, choices, checks, encounters, dialogue, and flavor;
- fixed HP and stat values within schema bounds;
- character and enemy spell/item composition from the engine catalog; and
- references to the built-in map, sprites, music, ambience, and SFX.

The engine controls authority, dice/combat behavior, effects, enemy random action
selection, movement, collision, timers, inventory, voting, UI behavior,
validation, compatibility, and resource enforcement. Content never executes
code.

Story choices declare a visible `default_choice_id` for abstention. The engine
owns ballot eligibility, tie resolution, and story-check actor selection.
Spells/items each reference one engine effect and one target type; assembling
character kits does not create multi-effect action scripts. Guard and modifier
replacement, item resolution, and full-inventory handling follow the contract.

## Validation ownership

The `content` crate owns the built-in content DTOs, loading from bytes, pure
validation, and the canonical checksum, so the client and server share one
schema. `game_core` consumes validated content and owns the rules. The
repository owns complete accepted examples and rejected fixtures, rather than
maintaining a second schema in this document. Startup structurally validates the
immutable built-in aggregate before admitting a session.

Schema validation:

- rejects duplicate and unknown fields;
- enforces portable lowercase identifiers and paths;
- resolves every story, encounter, effect, map, sprite, and audio reference;
- requires each story vote's default to belong to its nonempty offered choice
  set; validates check stats, single-effect spell/item definitions, target
  types, recipient selectors, and positive bounded magnitudes;
- bounds text, collections, transition chains, graph size, map dimensions, and
  decoded resource dimensions;
- rejects unreachable required nodes and non-terminating automatic chains;
- disables TMX external entities, external resources, and parser network access;
  and
- verifies the canonical content checksum.

Exact ceilings freeze with typical, large, limit, and rejected fixtures.
Released limits do not change silently.

## Release completeness

The [release minimum](mvp-contract.md#release-content-minimum) is a separate gate:

- Inventory checks count the required characters, enemies, spells, items, maps,
  localized text, and packaged audio/assets.
- Graph checks and headless route transcripts verify presented encounters,
  checks, choices, and dialogue, and that surviving routes cannot bypass the boss.
  Record the exercised branches and conditions; one happy-path run is not proof
  that every supported route meets its counts. Early wipes are not required to
  present the remainder of the route.
- Playtests measure actual run duration; `estimated_minutes` is authoring metadata.

Partial development aggregates pass structural validation without passing release
completeness. Phase 04 supplies fixtures distinguishing those outcomes; Phase 08
completes the inventory/route gate, Phase 12 qualifies timing, and Phase 13 repeats
the applicable checks on final packages.

## Map and asset direction

Formats belong to [the contract](mvp-contract.md#map-and-asset-formats). Source
maps embed tileset/collision data in TMX instead of leaving runtime TSX imports.
Image references name validated packaged assets; they cannot cause parser file
or network fetches. Phase 03 fixtures prove collision/event objects survive export.

Presentation fallback never changes gameplay authority, collision, controls,
focus, timing, or events.

## Ownership boundary

The runtime, built-in assets, schema code, fixtures, and built-in content are All
Rights Reserved in `perissables`.
