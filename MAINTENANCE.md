# Mercury (grave0x maintainer fork) — MAINTENANCE NOTES

Maintained fork of **Alex313031/Mercury** — "Mercury Browser", a Firefox fork with
compiler optimizations and patches from LibreWolf, Waterfox, Ghostery, and BetterFox.
License: MPL-2.0.

## Layout

| Role      | Remote | URL |
|-----------|--------|-----|
| origin    | this fork (grave0x) | https://github.com/grave0x/Mercury |
| upstream  | original project    | https://github.com/Alex313031/Mercury |

## What this repo is

This repo is a **patch overlay + build tooling** repo (~14 MB), not the Firefox source.
Workflow: `bootstrap.sh` fetches the Mozilla tree, `setup.sh` copies Mercury files over
it, `build.sh` builds. Docs: `docs/BUILDING.md`, `docs/PATCHES.md`, `docs/BUGS.md`,
`docs/DEBUGGING.md`, `TODO.md`.

## Upstream status (checked 2026-08-23)

- Last release: **v.129.0.2** (2024-09-17); last `main` commit 2024-12-03.
- 1540 stars, 42 forks, 91 open issues — project is effectively dormant.
- `pushed_at` 2026-08-09 shows ref activity, but no new commits/releases since Dec 2024.
- Only branch: `main`. Tags: v.112 … v.129.0.2.

Adoption rationale: upstream is dormant → maintainer fork keeps it alive: rebase to a
current Firefox ESR, triage the 91 open issues, and rebuild/package releases.

## Sync

`~/Projects/bin/sync-mercury` fetches upstream, fast-forwards local `main`, pushes to
origin, and mirrors tags. Run `sync-mercury --check` for a status-only look.

## Build constraints (grave's machine)

- The repo itself builds nothing; a real build needs the Mozilla tree
  (`bootstrap.sh`) plus an objdir — typically **100+ GB disk, 16+ GB RAM, hours**.
- Current machine state: /home at 88% (≈57 GB free), ~15 GB RAM total, baseline
  ~50% CPU from other sessions. A full build here requires freeing disk and heavy
  throttling (`-j 2/4`, nice/ionice, background nohup) per /home/grave/AGENTS.md.
- Recommended: do builds on a spare box / VM / after cleanup, not on this desktop.

## Security

- **CVE-2024-9680 (CVSS 9.8, exploited in the wild)** affects the 129.0.2 base
  (use-after-free in Animation timelines; fixed in Firefox 131.0.2 / ESR 128.3.1).
  The ESR rebase (T-0047) is therefore P0, not routine. Do not ship any 129.x build.

## Reports (committed)

- `reports/triage-2026-08-23.md` — 92 open issues / 0 PRs; 15 actionable, 42
  likely-duplicate (11 groups), 35 stale. Top actionable list included.
- `reports/porting-list.md` + `reports/porting-diff-summary.json` — per-file diff
  of the overlay vs ESR 153.1.0, with the newtab -> `browser/components/topsites`
  refactor callouts and a 6-step porting order.

## Backlog (candidate maintenance work)

- [ ] Rebase patch set onto current Firefox ESR (129 → 140.x) — main effort.
- [ ] Triage 91 upstream issues; close stale/duplicates, document answers.
- [ ] CI: build+package workflow (upstream has none visible).
- [ ] Re-run/refresh `version.sh`, `tot.sh`, `revert.sh` workflow against ESR.
- [ ] Decide Windows/macOS packaging scope (scripts exist: make_deb.sh, package.sh).
