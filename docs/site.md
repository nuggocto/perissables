# Public website

Authority: product boundaries come from `docs/mvp-contract.md`.

## Scope

One static marketing page for Les Périssables, plus a changelog and news feed.
Nothing else ships on the web.

Sections:

- game presentation — premise, characters, screenshots, trailer;
- changelog — released versions, newest first;
- news — short posts about updates and release milestones;
- Steam link — the store page is the only call to action; and
- footer — legal, privacy, and support contact.

## Shape

- Static HTML/CSS built from repository-owned sources; no application server and
  no database.
- Changelog and news are Markdown files in this repository. Changelog entries
  publish with each game release; news posts can publish at any time, including
  before launch.
- The site goes live with the Steam "Coming Soon" page after Phase 06 and is
  finalized in Phase 13.
- Content stays consistent with the current Steam store copy.
- The site never talks to the game server and is never required for gameplay.
