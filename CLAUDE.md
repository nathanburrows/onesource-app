# CLAUDE.md

## Repo shape

This repo contains a single file: `index.html`. It's the renderer mirror of the
upstream `nathanburrows/OneSourceCMD` Electron app. Most commits here are
auto-generated "sync renderer from OneSourceCMD@<sha>" commits that overwrite
the file wholesale. Direct edits made here should also be applied upstream
(or they'll be clobbered on the next sync).

## Version timestamps — bump on EVERY direct edit

Four strings in `index.html` display the build timestamp in the UI. On every
direct edit to `index.html`, update all four to the current Eastern Time using
the format `Mon D · H:MM AM/PM ET` (e.g. `May 18 · 12:01 PM ET`).

Locations (line numbers drift; grep to find them):

- `gs-brand-ver-ts`   — global sidebar header
- `upload-brand-ver-ts` — upload screen
- `ds-brand-ver-ts`   — data sources screen
- `page-brand-ver-ts` — page header

One-shot bump:

```sh
NEW="$(TZ='America/New_York' date '+%b %-d · %-I:%M %p ET')"
OLD="$(grep -oE '[A-Z][a-z]+ [0-9]+ · [0-9]+:[0-9]+ [AP]M ET' index.html | head -1)"
sed -i "s/$OLD/$NEW/g" index.html
```

Verify with `grep -n 'brand-ver-ts' index.html` — all four should show the same
new timestamp. The upstream sync process bumps these too, so keeping them
current means the visible version stamp matches the deployed code.

## Git workflow

- Develop on the branch the harness assigns (e.g. `claude/<task>-<id>`).
- Push with `git push -u origin <branch>`.
- Don't open PRs unless explicitly asked.
