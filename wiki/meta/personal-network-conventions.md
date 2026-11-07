---
type: meta
title: "Personal Network Conventions"
created: 2026-11-08
updated: 2026-11-08
status: evergreen
tags:
  - meta
  - conventions
  - personal-network
related:
  - "[[Meta Index]]"
  - "[[engineering-conventions]]"
---

# Personal Network Conventions

Optional layer for vaults that also hold the user's **personal life** (people, groups, events, trips, places) next to the work-oriented engineering layout. It is independent of projects: nothing in `wiki/projects/` depends on it, and a vault that never creates `wiki/life/` is unaffected.

Navigation: [[Meta Index]] | [[engineering-conventions]] | `life-network.base` (dashboard)

## Layout

```
wiki/life/
  people/      one note per person          (template: life-person)
  groups/      recurring groups of people   (template: life-group)
  events/      parties, outings, weekends   (template: life-event)
  travel/      trips                        (template: life-event)
  places/      meaningful places            (template: life-place)
```

Work contacts and companies stay in `wiki/entities/`; personal people live here. The `life-network.base` view lists everything under `wiki/life/` that is not a `meta` page, grouped by type, plus a timeline by year.

## Frontmatter

| Field | Used by | Meaning |
|---|---|---|
| `type` | all | `family \| friend \| acquaintance \| colleague` for people, otherwise `group`, `event`, `place` |
| `relationship_status` | people | `confirmed \| probable \| candidate` |
| `confidence` | events, places | `confirmed \| probable \| candidate` |
| `participants`, `participants_status` | events | who was there, and whether the roster is `complete`, `partial` or `unknown` |
| `groups`, `members`, `events`, `people` | all | wikilinks that connect the layer |
| `sensitive` | all | `true` for anything about people; keep the vault private |

## Certainty rules

- **confirmed** — the user said so, or at least two independent sources agree (for example a photo album and a chat).
- **probable** — one good source or consistent circumstantial evidence.
- **candidate** — a hint only. Never present a candidate as a fact.
- A presence is recorded only if it is confirmed or probable. Being a member of a chat is **not** evidence of attending an event.
- Do not infer people from faces in photos.
- Keep a single queue page (for example `wiki/life/Candidates to confirm.md`) with open questions, and clear it in batches with the user instead of interrupting every session.

## Privacy

Keep out of the vault, even in a private one: phone numbers, e-mails, street addresses of other people, health data, money matters of third parties, and private conflicts. Keep originals, transcripts and media outside the vault and link to a source note that only holds an index and a short description.

Use `sensitive: true` on every person, group and event note and never copy this layer into a public repository.
