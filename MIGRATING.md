# Migrating to v2.0.0 (Engineering Layout)

v2.0.0 restructures the wiki folder layout. **Your notes are not lost** — a
migration engine moves them. This guide covers the normal upgrade and the
recovery path if `/sync` gets stuck on the big structural jump.

> Your vault content is committed **locally only** (fetch-only remote), so your
> local git history is disposable — only the *files* matter. That makes the
> recovery below safe.

---

## A. Normal upgrade (small gap, or a fresh clone)

Run on your work laptop, in order:

```bash
bash bin/sync.sh                        # pull v2.0.0 machinery (content is kept — local wins)
bash bin/setup-mode.sh --mode engineering   # mode.json is local-wins on sync; set it explicitly
bash bin/migrate-structure.sh           # DRY RUN — review the move plan
bash bin/migrate-structure.sh --apply   # loss-safe: snapshots, git mv, verifies zero loss, never pushes
bash bin/setup-vault.sh                 # refresh Obsidian graph + machinery-hide filters
cp -r skills/. ~/.opencode/skills/tablinum/   # refresh installed skills (or the ~/.claude path)
/wiki-lint                              # confirm no dead links
bash bin/setup-dragonscale.sh           # only if you use DragonScale
```

### Three things `/sync` can't auto-fix (it makes *local* win for these)
1. **`.vault-meta/mode.json`** — your old `mode: para` survives the sync. Run
   `bash bin/setup-mode.sh --mode engineering` (step 2 above).
2. **`.obsidian/`** (gitignored, machine-local) — new graph colours + the
   "hide machinery from file explorer" filters only land via
   `bash bin/setup-vault.sh` (step 5).
3. **Installed skills** — the copies under `~/.opencode/skills` /
   `~/.claude/skills` are stale until you re-copy them.

---

## B. Recovery — `/sync` stuck mid-rebase with many conflicts

If your vault has a lot of existing content, `/sync`'s rebase replays your
content-commits across every structural commit in v2.0.0 and can drown in
rename/content conflicts (and time out). Symptoms: "rebase in progress",
hundreds of conflicts, local main N commits ahead / origin/main many commits
ahead.

**Don't panic and don't `git reset --hard` yet.** Do this instead:

### 1. Unstick (no data loss)
```bash
git rebase --abort 2>/dev/null; git merge --abort 2>/dev/null
git status        # should be clean; your old-structure content is back
ls wiki/          # your folders + notes present
```
`--abort` restores the exact pre-sync state.

### 2. Back up content OUTSIDE the repo
```bash
cp -r wiki ~/tablinum-content-backup-wiki
cp -r .raw ~/tablinum-content-backup-raw 2>/dev/null
```

### 3. Take v2.0.0 cleanly
`.obsidian/` is gitignored (untouched); your content is backed up.
```bash
git fetch origin
git reset --hard origin/main
```

### 4. Restore your content on top (lands in the old paths)
```bash
cp -r ~/tablinum-content-backup-wiki/. wiki/
cp -r ~/tablinum-content-backup-raw/. .raw/ 2>/dev/null
```

### 5. Migrate old paths → new layout
```bash
bash bin/migrate-structure.sh            # DRY RUN — read the plan
bash bin/migrate-structure.sh --apply    # git mv, merge-on-collision, verifies zero loss
bash bin/setup-vault.sh                  # refresh Obsidian graph + machinery-hide
/wiki-lint                               # catch any dead links
```

After step 3, `mode.json` is already `engineering` and the structure marker is
absent, so migrate runs the full v1→v13 chain. No `setup-mode` needed.

### Caveats
- The above is fully safe **if your local commits are all `wiki/` notes**. If
  you also edited machinery (scripts/skills/templates) locally, copy those
  changes aside in step 2 too — `git reset --hard` will replace them with
  v2.0.0's versions.
- Keep the step-2 backup until `migrate --apply` reports success and
  `/wiki-lint` is clean. Then delete it.

---

## What the migration guarantees

`bin/migrate-structure.sh --apply`:
- snapshots to a commit before touching anything,
- moves with `git mv` (history preserved), **merges on collision** (never
  overwrites),
- verifies the markdown page count does not drop, and **hard-resets to the
  snapshot** if anything would be lost,
- **never pushes**.

See `CHANGELOG.md` for the full list of what moved where.
