# Mercury — Rebase Plan (129.0.2 → current Firefox)

Maintainer: grave0x. Generated 2026-08-23. Scope: revive the dormant upstream by
rebasing the Mercury patch overlay onto a supported Firefox base.

## 1. Current state (as adopted)

- Upstream pinned base: mozilla-unified changeset `c6f0209c79239408bef9b3c98e9c729dcf20ec0c`
  = Firefox **129.0.2** era (2024-08-16). Set in `version.sh` (`MERCURY_BRANCH`).
- Overlay surface: **107 files, ~5.5 MB** copied over the Mozilla tree by `setup.sh`.
- Last upstream release: v.129.0.2 (2024-09-17). No commits/releases since Dec 2024.
- Build toolchain: `bootstrap.sh --linux` (mozboot → mozilla-unified),
  `version.sh` (pin changeset), `setup.sh` (apply overlay), `build.sh` (`mach build`),
  `package.sh` / `make_deb.sh` (artifacts). `tot.sh` tracks central; `revert.sh` resets.

## 2. Target bases (Mozilla versions, 2026-08-23)

| Channel | Version | Notes |
|---------|---------|-------|
| ESR (current) | 140.14.0esr | Stable; stepping-stone target. Goes EOL at ESR transition (~Oct 2026). |
| ESR (next)    | 153.1.0esr | The long-lived ESR once it replaces 140. **Final recommended target.** |
| Release       | 154.0     | Latest stable; faster cadence, more frequent rebase work. |
| Nightly       | 156.0a1   | Not a maintenance target. |

**Recommendation**: first rebase to **ESR 140.14.0** to validate toolchain + overlay
fixes at a small jump (129 → 140), then immediately rebase again to **ESR 153.1.0** as
the sustained ESR line. Releasing on 154 (release) is an optional faster-moving track.

## 3. Rebase mechanics

1. Pick the target changeset. Resolved mozilla-unified nodes (2026-08-23):
   - **ESR 153.1.0** (recommended sustained target): `bdb74c45c2e1e3fe593fbf3c5ca6d2ab2046ef08`
   - **Release 154.0**: `9ce1ee6baeb9a3c326dbd180bdece65d8fc2eadc`
   - ESR 140.14.0 (stepping stone): tag resolves in `mozilla-esr140`, not
     `mozilla-unified` (release-branch repo). Optional intermediate.
   Set `MERCURY_BRANCH=<node>` in `version.sh`; `mozilla-unified` clone serves all.
2. Set `MERCURY_BRANCH=<changeset>` in `version.sh` (keep `mozilla-unified` as the
   `HG_SRC_DIR` clone; unified serves all branches).
3. `./bootstrap.sh --linux` (first time only) → `$HOME/mozilla-unified`.
4. `./version.sh` (pull, `hg update --clean -C <changeset>`, `./mach bootstrap`).
5. `./revert.sh` then `./setup.sh [flags]` to apply the overlay onto the new base.
   - Fix every overlay file that no longer applies cleanly (see §4).
6. `MOZ_MAKE_FLAGS="-j2" ./build.sh` (throttled — see §6), then `./package.sh`.
7. Validate per §5, then tag `v.<version>` in this repo and push.

## 4. Overlay risk matrix

| Area | Count | Rebase risk | What must be reconciled |
|------|-------|-------------|-------------------------|
| `browser/branding/mercury/*` + branding-common.mozbuild | ~60 | Low | Logos/icons/installer art carry over; check `.nsi`, `configure.sh`, `brand.*` strings still parse. |
| `browser/components/customizableui/CustomizableUI.sys.mjs` | 1 | **High** | UI patch ("ESR78 top bar", home/dev buttons). Upstream file changes almost every cycle. |
| `browser/components/newtab/lib/DefaultSites.sys.mjs` + `tippytop/top_sites.json` + favicons | 4 | **High** | Pocket/sponsored/highlights removal + custom top sites. Newtab internals churn. |
| `netwerk/protocol/http/nsHttpHandler.cpp` | 1 | **High** | Telemetry/privacy/UA patch in HTTP handler. C++ — must merge by hand. |
| `app/profile/firefox.js`, `app/nsBrowserApp.cpp` | 2 | Medium | DNT/GPC defaults, pref blocks, app init changes. |
| `browser/base/content/default-bookmarks.html` | 1 | Low | Bookmark set; additive. |
| `build/moz.configure/lto-pgo.configure`, `toolchain.configure` | 2 | **High** | AVX/AES/LTO/PGO flag patches against mozconfig system — flag names drift. |
| `toolkit/moz.configure`, `browser/moz.configure`, `confvars.sh`, `application.ini(.in)`, `codename.txt` | 5 | Medium | JXL enable, hardening, branding dir, codename `hydrargyrum`. |
| `policies/policies.json` + `moz.build` | 2 | Medium | Enterprise policy (unsigned extensions, telemetry off). JSON schema may drift. |
| `testing/profiles/profileserver/user.js`, `devtools/*.svg`, `ipc/app/module.ver`, `other-licenses/7zstub/*`, installer `.nsi`/`.tag` | ~25 | Low | Mostly art/version strings. |
| `mozconfigs/*` (17 variants) | 17 | Medium | Review each against current mozconfig options (AVX2/SSE3/SSE4/Win/Mac/arm64). |

The ~10 **High** files are the real rebase work; the rest is mostly copy-forward + string
refresh (version → 140/153, brand name stays Mercury).

## 5. Validation checklist

- [ ] `./mach build` completes with `-j2` (no overlay-related compile errors).
- [ ] `./mach run` launches; `about:` shows correct version + Mercury branding.
- [ ] Telemetry is off; DNT/GPC on; Pocket/highlights/suggested content absent.
- [ ] Unsigned extensions installable (policies.json).
- [ ] JPEG XL decodes (about:config `image.jxl.enabled`).
- [ ] Package artifacts: `package.sh` tarball, `make_deb.sh` .deb, Win installer.
- [ ] Tag release `v.<version>` and publish to grave0x/Mercury Releases.

## 6. Machine constraints (grave's desktop, per /home/grave/AGENTS.md)

- Full Firefox build needs ~100+ GB disk + 16+ GB RAM + hours. This machine: ~57 GB
  free, ~15 GB RAM, baseline ~50% CPU. **Do not build here unthrottled.**
- Preferred: build on a spare box / Arch VM (see vm-feature-testing skill), or after
  freeing disk; always `-j2`/`-j4`, `nice -n 19`, `ionice -c3`, backgrounded nohup + log.
- The overlay prep (diffing/porting ~10 high-risk files) is light and can be done here.

## 7. First concrete steps (read-only, this machine)

1. Fetch `mozilla-esr140` / `mozilla-esr153` release-tag changesets.
2. Diff each High-risk overlay file against its 140/153 counterpart
   (GitHub mozilla-unified / hg.mozilla.org raw) to produce a per-file porting list.
3. Refresh `version.sh` pin + version strings; commit.
4. Attempt the actual build only on an approved heavy-build environment.
