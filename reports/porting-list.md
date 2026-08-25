# Mercury overlay porting list — 129.0.2 → ESR 153.1.0

Generated 2026-08-23 (read-only diff of overlay files vs mozilla-unified @
`bdb74c45c2e1e3fe593fbf3c5ca6d2ab2046ef08`, i.e. FIREFOX_153_1_0esr_RELEASE).

Overlay base (current): `c6f0209c79239408bef9b3c98e9c729dcf20ec0c` (Firefox 129.0.2).

## Summary

~10 files carry real code/config changes and must be hand-ported. The rest are
branding/version strings (copy-forward + string refresh). Three overlay files
have **no direct upstream counterpart at 153** — resolved in §2.

## 1. Per-file diff (high-risk only)

| Overlay file (repo) | Upstream counterpart @153 | Diff | Verdict |
|---|---|---|---|
| browser/components/customizableui/CustomizableUI.sys.mjs | same | +2753/-609 (5062 ln) | HIGH churn. UI patch (ESR78 top bar, home/dev buttons) must be re-applied by hand. |
| app/profile/firefox.js | browser/app/profile/firefox.js | +1084/-709 (2945 ln) | HIGH. Pref blocks (telemetry, DNT/GPC, Pocket) drifted heavily. |
| toolkit/moz.configure | same | +975/-364 (1967 ln) | HIGH. JXL/hardening flags + mozconfig drift. |
| build/moz.configure/toolchain.configure | same | +532/-574 (1768 ln) | HIGH. AVX/AES/LTO/PGO flag code moved around. |
| netwerk/protocol/http/nsHttpHandler.cpp | same | +596/-329 (1712 ln) | HIGH. C++ hand-merge of privacy/telemetry patch. |
| app/nsBrowserApp.cpp | browser/app/nsBrowserApp.cpp | +163/-78 (399 ln) | MEDIUM. Re-apply app-init patch. |
| build/moz.configure/lto-pgo.configure | same | +64/-34 (203 ln) | MEDIUM. Port LTO/PGO flags. |
| browser/base/content/default-bookmarks.html | same | +22/-12 (59 ln) | LOW. Merge bookmarks list. |
| browser/moz.configure | same | +6/-7 (40 ln) | TRIVIAL. |
| browser/confvars.sh | same | +1/-1 (9 ln) | TRIVIAL (branding dir already set). |

## 2. Files with no upstream counterpart at 153

- **browser/components/newtab/lib/DefaultSites.sys.mjs** — GONE. The newtab
  `lib/` tree was split into a new component **`browser/components/topsites/`**
  (contains `TopSites.sys.mjs`, `TippyTopProvider.sys.mjs`, `constants.mjs`,
  `content/`, `jar.mn`, `moz.build`, `test/`). Mercury's custom DefaultSites
  (removes amazon/sponsored sites) must be ported into `TopSites.sys.mjs`.
  (Upstream removal: bug 1938452, ~Fx 136.)
- **browser/components/newtab/data/content/tippytop/top_sites.json** — GONE.
  TippyTop's checked-in JSON was replaced by remote-settings data served by
  `browser/components/topsites/TippyTopProvider.sys.mjs`. Mercury's custom
  top_sites.json needs a new mechanism (patch the provider or ship a custom
  remote-settings override).
- **policies/policies.json** — NOT a rename; this is a Mercury-*added* dir
  (`setup.sh` does `mkdir -p ${HG_SRC_DIR}/policies && cp -r ./policies/.`).
  It has no upstream counterpart, so it carries over unchanged. Only re-check
  that the policy schema still applies at 153 (enterprise policy keys drift
  slowly; `browser/components/enterprisepolicies/schemas/` is the schema source).

## 3. Recommended porting order

1. Toolchain/mozconfigs (`toolchain.configure`, `lto-pgo.configure`,
   `toolkit/moz.configure`, `browser/moz.configure`, `confvars.sh`) — unblock the
   build system first.
2. `firefox.js` + `policies.json` (prefs/policy) — behavioral core.
3. `nsHttpHandler.cpp` + `nsBrowserApp.cpp` (C++ hand-merge).
4. `CustomizableUI.sys.mjs` (UI patch, largest churn).
5. Newtab → `topsites` port (DefaultSites → TopSites.sys.mjs; tippytop → new provider).
6. Branding/version strings (copy-forward, update version numbers).
