# Changelog

All notable changes to **tablinum** (CoderVinz fork). This fork versions
independently from upstream.

---

## [Unreleased]

### Added
- **Optional personal-network layer** for vaults that also track personal life:
  templates `life-person`, `life-group`, `life-event`, `life-place`, the
  `life-network.base` dashboard (table of people/groups/events/places plus a
  timeline by year) and `wiki/meta/personal-network-conventions.md` (layout,
  frontmatter, certainty levels `confirmed | probable | candidate`, privacy
  rules). Nothing changes for vaults that never create `wiki/life/`.
- `portfolio` frontmatter field on projects (`main-work | side-work | personal`,
  optional) and a grouped *Project Portfolio* view in `engineering.base`.
- Guidance for an "Access and infrastructure" note (no secrets, only the name of
  the password-manager entry) in `engineering-conventions.md`, and a *Remotes*
  section in `AGENTS.md` (`upstream` vs `origin`).

### Changed
- **Convention proposal:** the project MOC is named after the project
  (`projects/<slug>/<Project Name>.md`) instead of `_project.md`, so wikilinks are
  unambiguous and the graph view shows readable labels. Templates, `SETUP.md`,
  `AGENTS.md` and the conventions doc are updated accordingly. Existing
  `_project.md` notes keep working; rename them at your own pace.
- `engineering.base`: sorted People view, project views show `title` and keep the
  `deadline` column.

### Fixed
- `scripts/tiling-check.py`: pages longer than the embedding model's context no
  longer fail with `embed_error`. Long pages are split on paragraph boundaries,
  embedded in chunks (retrying with halves on failure) and averaged by length.
  Short pages keep the original single-call path.
- `scripts/tiling-check.py`: replaced deprecated `datetime.utcnow()` with
  timezone-aware UTC; the generated report now starts with the standard
  frontmatter (`type: meta`, `status: developing`) so it passes lint.
- Shipped meta pages (`dashboard`, `getting-started`, `methodology-modes`,
  `transport-fallback`, `engineering-conventions`) were missing
  `created`/`updated`/`tags` frontmatter and failed lint.

---

## [2.0.0] — 2026-07-24 — "Engineering Layout"

The vault gets a purpose-built, self-migrating layout, its own first-class
organizational mode, and a loss-safe migration engine. This is a structural
release: the wiki folder layout changed a lot — but **nothing is lost**, a new
migration tool moves your notes for you.

### ⚠️ Upgrade — do this first (testers)

> **Full step-by-step + recovery if `/sync` gets stuck: see [MIGRATING.md](MIGRATING.md).**

On your work laptop, in order:

1. **`bash bin/sync.sh`** (or `/sync`) — pulls the update. It now prints
   `structure: layout advanced vN -> v13` when your content is behind.
2. **`/migrate`** (or `bash bin/migrate-structure.sh`) — shows a **dry-run
   plan** first, asks you to confirm, then moves content with `git mv`
   (history preserved), **merges on collision** (never overwrites), and
   **verifies zero page loss** before committing. It never pushes. If anything
   would drop a page, it hard-resets to the pre-migration snapshot and aborts.
3. **`bash bin/setup-vault.sh`** once — refreshes the Obsidian graph +
   file-explorer config for the new folders.
4. If you use DragonScale: **`bash bin/setup-dragonscale.sh`** (unchanged).

Migrations chain, so a vault on any older version catches up in one `/migrate`.

### ✨ Headline

- **New `engineering` mode** — tablinum's layout is now its own first-class
  methodology mode (alongside Generic / LYT / PARA / Zettelkasten), not a
  mislabeled PARA. The four reference modes are restored to pristine vanilla;
  `engineering` is the shipped default. Set/inspect via `bash bin/setup-mode.sh`.
- **Versioned, loss-safe migration engine** — `bin/structure/schema.json` is the
  single source of truth for the layout (`version` + declarative `renames[]`);
  `bin/migrate-structure.sh` + `/migrate` apply changes safely; `/sync` prompts
  you when the layout advances. Structural changes made upstream now propagate
  to your laptop and migrate existing pages without data loss.
- **Redesigned vault layout** — 15 folders down to 11 top-level, each with one
  clear, non-overlapping purpose.

### 🗂 Layout changes — what moved (the migration does this for you)

| Before | After | Why |
|--------|-------|-----|
| `entities/` (people, orgs, products, repos) | `entities/` = **people + companies/teams only** | one clean org map |
| `resources/tools/` | **`technologies/`** (top-level) | tech/stack inventory is first-class |
| `resources/snippets/` | **`snippets/`** (top-level) | reusable code stands alone |
| `resources/patterns/` | `concepts/` (`type: pattern`) | a pattern is a concept |
| `resources/glossary/` | `concepts/` (`type: glossary`) | a term is a concept |
| `questions/` | `concepts/` (`type: question`) | open questions live with concepts; dashboard still lists them |
| `comparisons/` | `concepts/` | a comparison is a concept |
| `references/` | `meta/` | the operating manual is meta, not a `resources` sibling |
| `areas/` | dissolved → **`operations/`** top-level | concrete folders over abstract PARA buckets |
| `design-system/`, `resources/design/` | removed | no design work in this vault |
| `processes/` | `operations/processes/` | operations is the "how things run" hub |
| `projects/inbox/` | **`sessions/`** (top-level) | sessions aren't projects; captures route by type |
| repos | `sources/` | repos are source material |
| `resources/` | dissolved | its contents got right-sized homes |

New folders: `technologies/` (+`docs/`), `snippets/`, `sessions/`,
`sources/research/`, `sources/incoming/`, `meta/lint-reports/`. Every folder
has a named `<Folder> Index.md` hub page.

### 🧭 Reconciled end-to-end

Every surface that references the layout was brought into agreement:

- **Router** (`scripts/wiki-mode.py`) — engineering routes `source→sources/`,
  `entity→entities/`, `concept→concepts/`, `session→sessions/`,
  `research→sources/research/`. Fixed a long-standing split-brain where the
  router pointed at folders that didn't exist.
- **`wiki-ingest` / `save` / `autoresearch`** routing updated to match.
- **Dashboards** (`engineering.base`, `dashboard.md`) — added a **Teams** view;
  Open Questions now filters by `type` (not folder); dead views removed.
- **Templates** — `process` gains `tools:` + `project:` links; `team` gains a
  queryable `members:` list; `person` `team:` clarified; paths updated.
- **Graph + file-explorer CSS**, **`wiki/index.md`** table + hub pages, all
  five per-agent instruction files (CLAUDE / AGENTS / GEMINI / Copilot).

### 🩹 Fixes

- `wiki-ingest` and `wiki-lint` no longer disagree on whether DragonScale
  addresses are active (both gate on the setup-created counter).
- `.gitattributes` forces **LF** on scripts — CRLF checkouts no longer break
  `bash`/`python` on WSL2.
- Removed dead references throughout: stripped-test-suite links, a nonexistent
  `wiki/domains/` in the CSS, an undefined `wiki/inbox/` in distill, dangling
  audit citations, and demo-vault placeholder links.
- `setup-mode.sh` PARA seeding restored to vanilla; `engineering` seeds the
  real layout.
- DragonScale `legacy-pages.txt` is now per-machine (correct rollout baseline
  instead of a stale shipped date).

### 📄 New files

- `bin/structure/schema.json`, `bin/migrate-structure.sh`, `bin/migrations/`
- `commands/migrate.md` (+ `.opencode/command/migrate.md`) — `/migrate`
- `.github/copilot-instructions.md`
- `CHANGELOG.md` (this file)

### 🧰 Known / deferred

- `technologies/docs/` and `meta/lint-reports/` are utility subfolders without
  their own hub pages (by design).

---

## [1.9.2] and earlier — foundation (pre-fork-overhaul)

The tablinum base this fork builds on: the LLM Wiki pattern (Karpathy),
Compound Vault (v1.7 — CLI transport, hybrid retrieval, per-file locking,
pre-commit verifier), methodology modes (v1.8), the `think` skill (v1.9),
the multi-project engineering layer, technology inventory, org map, and the
`bin/sync.sh` zero-touch pull. Machinery originates from
[AgriciDaniel/tablinum](https://github.com/AgriciDaniel/tablinum) (MIT); see
`ATTRIBUTION.md`.
