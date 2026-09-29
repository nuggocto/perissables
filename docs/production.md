# Production inventory

Status: Planning inventory
Owner: Sole developer

Product requirements and release counts belong to `docs/mvp-contract.md`. This
file tracks how approved scope becomes finished screens, content, assets, and
store deliverables. A `TBD` is a decision to make, not permission to invent
scope during implementation.

## Product flow and screens

```text
startup/connection
  -> lobby and character selection
  -> world exploration
  -> dialogue, choice, and check feedback
  -> combat
  -> run summary
  -> cleared lobby
```

Required supporting surfaces include settings/keybindings, reconnect/resync
progress, compatibility/update-required notices, and public/run-lost errors.
Exact layouts and overlays are approved before their implementation phase; rough
wireframes belong with the corresponding phase evidence rather than a second UI
specification here.

## Inventory

Exact counts and formats live in the [release minimum](mvp-contract.md#release-content-minimum)
and [asset contract](mvp-contract.md#map-and-asset-formats). This inventory tracks
production responsibility rather than duplicating those values.

| Area | Deliverable | Owner/source | Status |
| --- | --- | --- | --- |
| Stories and dialogue | Authored route and localized beats | Owner and credited friends | Planned |
| Maps and tilesets | Supermarket and storage-room exports | Owner, credited friends, purchased sources if used | Planned |
| Playable characters | Premade roster and directional frames | Owner and credited friends | Planned |
| Enemies and boss | Kits, authored loot lists, and art | Owner and credited friends | Planned |
| Spells, items, effects | Engine catalog and validated action data | Project | Planned |
| UI and icons | Scalable keyboard-operable set | Owner, credited friends, purchased sources if used | Planned |
| Animation and VFX | Presentation-only set; scope TBD after wireframes | Owner and credited friends | Planned |
| Music, ambience, SFX | Approved runtime audio exports | Owner, credited friends, purchased sources if used | Planned |
| Fonts and text | Packaged fallback fonts and reviewed launch locales | Owner, fluent credited friend, licensed fonts | Planned |
| Steam store media | Capsule art, screenshots, trailer, copy; initial and final sets | Owner and credited friends | Planned |

Remaining `TBD` values are locked before production work in that area is
estimated or accepted.

## Asset pipeline and naming

- Preserve editable source files and record the tool/version needed to export
  them; runtime assets contain only approved exports.
- Every friend contribution needs written permission or assignment covering game
  and Steam distribution, modification, credit, and continued use. Friendship or
  a verbal agreement is not provenance.
- Runtime IDs and paths follow the portable lowercase rules from the content
  contract. One manifest maps each shipped asset to its source, creator,
  license, approval, and export settings.
- Prefer reproducible command-line exports when the source tool supports them.
  Otherwise record the manual export steps and review the resulting checksum.
- Keep temporary, source-tool, and unlicensed files out of release packages.
- The approved font set must render English and French, including French
  diacritics in the title, keyboard prompts, and fallback glyphs, without
  network access.

## Acceptance

An asset or content item is complete only when:

- its owner/source and license or employment ownership are recorded;
- its runtime format, dimensions, duration, and decoded resource use pass the
  applicable validator limits;
- it works in the shipped entrypoint at required resolutions and UI scales;
- missing or failed presentation uses the documented safe fallback;
- its source and exported checksum are recorded; and
- no placeholder or debug asset remains in release features/packages.

## Early playtest record

Character names, HP/stat sheets, spell assignments, and enemy/boss kits and loot
lists remain Phase 05 content work. Enemy kits may be spell-only. Their
final values must be recorded before Phase 08 asset/content production begins.

Each scheduled playtest records the build/content revision, participant relation
to the project, party size, run duration, abandoned runs, rules questions,
observed idle time, confusing choices, defects, and whether participants wanted
to replay. Phase gates use the smallest useful session and do not invent a
numeric fun score before baseline evidence exists.

## Non-code release inventory

The Steamworks parallel track secures app access early. After Phase 06, a
first Steam "Coming Soon" page and the public site (`docs/site.md`) go live
with real screenshots after the initial publication checklist passes. Phase 12
updates those materials and assigns owners for final capsule art, screenshots,
trailer, store copy, age/content disclosures, launch languages, pricing proposal,
privacy/support contacts,
incident handling, server cost, and shutdown policy. Phase 13 approves and
publishes the final set against the exact release artifacts.
