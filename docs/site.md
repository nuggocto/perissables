# Public website

Authority: product boundaries and publication prerequisites come from
[the contract](mvp-contract.md#supported-platforms-and-public-website). This file
owns page organization and the publication checklist, not new product scope.

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

## First publication checklist

Before the Phase 06 site or Steam "Coming Soon" page goes public:

- Approve accurate copy, launch-language presentation, and media rights/credits.
- Publish legal identification and working privacy/support contacts.
- Record the selected static host and its actual logging/retention behavior;
  publish a matching privacy notice. Prefer linked media; any embed or cookie
  must be reflected in the notice and applicable consent behavior.
- Check navigation, keyboard access, Steam links, and page readability on the
  built site. The privacy/support information must be reachable.

The site adds no analytics or tracking by default. Phase 13 updates and rechecks
the published material against the release; approval is not deferred until then.
