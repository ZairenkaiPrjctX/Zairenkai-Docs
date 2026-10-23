# Implementation roadmap

Phases from the ecosystem master prompt, with the state reached so far. A box is ticked only
when its acceptance check has output recorded in `audit/ECOSYSTEM_AUDIT.md` or a linked report.
The **session checkpoint** at the end is rewritten at every session close.

## Phase 0 · Inventory and audit

- [x] Clone the 16 in-scope repositories; anime-extensions excluded
- [x] Heads, visibility and empty-repository facts reconciled (`audit/ECOSYSTEM_AUDIT.md` §1)
- [x] Each repository's own checks run where the toolchain exists (§2)
- [x] The eight audit leads reproduced (§3)
- [x] Six documents: audit, architecture, responsibility matrix, contracts, roadmap, threat model
- [ ] `.github` inventory, generator and validator audited (blocked: access, F-08)
- [ ] Upstream catalog audited for graphics (F-15; done for Tuning and local AI)

Acceptance: findings carry severity and evidence class; no device claim without hardware.

## Phase 1 · Unblock reliable integration

- [x] App CI and release fetch public locked SHAs without a token (App `e0d53bb`)
- [x] Modules CI likewise (Modules `298311c`)
- [x] App inventory reconciled: 17 repositories, Tuning added, anime-extensions excluded
- [x] Stale docs fixed in App, Core, FAS, ART, ThermalTools, Database-Source
- [x] Release doc reconciled with the trust list (App `d0db6b7`)
- [x] **App full-integration job green on GitHub** without `allowMissingComponents` (run
      39827139485 at `d0db6b7`; again 39830066001 at `e1ca387`)
- [x] Modules `module` job green on GitHub (run 39827725469)
- [ ] Release gate re-run on the full-integration APK; output recorded
- [ ] Central `.github` inventory updated (needs access)

Acceptance: a CI run id with every full-integration step green, and the integration report.

## Phase 2 · Contracts and module authority

- [x] ADR in Zairenkai-Sdk: producers canonical, SDK mirrors and validates (Sdk PR #1, proposed)
- [x] SDK schemas for Core v1, FAS, zthermal, zart, AFID, tuning; tests against captured fixtures (25 tests, 33 fixtures)
- [x] SDK-defined schemas: module catalog, Local-AI types (entitlements still open)
- [ ] Catalog generation moved from App to Modules, with old/new catalogs compared byte for byte
- [x] App re-pinned to Modules with ztune and to Core with the tuning adapter (App `e1ca387`, CI green)
- [x] Typed tuning client in the app over Core's `tuning` method (App PR #6, CI 39888821755; not merged)

Acceptance: SDK CI validates every captured fixture; App full-integration green at the new pins.

## Phase 3 · Empty repositories, one at a time

- [x] Intelligence-Engine spike (IE PR #1): ADR LiteRT-LM vs llama.cpp, one offline model load,
      typed tool calls through a closed `ToolRegistry`, airplane-mode proof on a Linux host; no
      remote AI API. Eval 9/16: the 0.5B model is too weak alone
- [ ] Intelligence-Engine Phase B: runtime comparison on Android devices (needs devices)
- [x] Graphics-Platform: ADR on stack (proposed); read-only Vulkan probe and GLES/EGL probe as separate
      subsystems, host-tested on Mesa lavapipe/llvmpipe (Graphics-Platform PR #1, CI 39888606008; not merged)
- [ ] Graphics-Platform: first read-only run on a phone; App integration (`libzgfx.so`); SDK mirror of the contract
- [ ] Tuning: maintainer decides ADR 0001 (executor), creates `dev`; first Tier D profile only with hardware evidence
- [ ] Release: orchestration over verified inputs, SBOM, notices, checksums
- [ ] License-Server: scope ADR; offline-safe; optional
- [ ] Docs site build and ZairenkaiPrjctX.github.io with link and accessibility checks

Acceptance per repository: build, lint, unit and contract tests in CI; no README-only repositories.

## Phase 4 · App integration

- [ ] Every screen backed by a typed contract and a real adapter
- [ ] Denial, missing component, timeout, malformed JSON, schema mismatch, old version, no root manager, cancellation tests
- [ ] Captured golden fixtures from current binaries

## Phase 5 · Verification and release readiness

- [ ] Every repository's documented checks green in CI
- [ ] Signed APK and catalog from `release.yml`; gate passes
- [ ] Physical-device validation records (Core-only first, then FAS, then module lifecycle)

The product is not release-ready until all of Phase 5 holds.

---

## Session checkpoint · 2026-10-23

**Phase:** 2 nearly done (catalog move and SDK consumers open), 3 in progress. **Last completed
task:** Graphics-Platform `zgfx` (read-only Vulkan and GLES/EGL probes) and its first green CI.

### Merged since the last checkpoint (maintainer)

Core #3 (`345fc7f`), FAS #4 (`1c453db`), Modules #1 (`bee45bd`), App #5 (`abff7d5`), Sdk #1
(`ed44c90`), Intelligence-Engine #1 (`cc3d48c`), Docs #1 (`9bd8833`), Database-Source #14 and #13.

### Changes in this session (branch `ccr-63f8aa87-1b0paj`)

| Repository | Commits | PR | CI |
|---|---|---|---|
| Zairenkai-Modules | `992751f` (pins Core and FAS merge commits) | draft #2 | green (39885361175) |
| Zairenkai-App | `cf6fdb8`, `fbe9e44`, `7f942a4` | draft #6 | green (39888821755); the first run (39887060148) failed on a uid-1001 capture, fixed in `fbe9e44` |
| Zairenkai-Graphics-Platform | `faf8870`, `b319e28` | draft #1 | green (39888606008), all 5 jobs including Mesa host probes |
| Zairenkai-Docs | this update | draft (to open) | no CI |

Every commit carries an SSH signature (`gpgsig` on the commit object).

### Commands and results

- App: `:integration-core:test` 99 passed; `:app:testDebugUnitTest` 29 passed; lint 0 errors;
  `verify_integration.py lock`/`build` ok; verifier tests 17; shellcheck clean;
  `build-components.sh` and `capture-contracts.sh` exit 0 (as root, and the tuning capture as
  uid 1001 via `runuser`). Gradle worked locally this time (no HTTP 429).
- Graphics-Platform: 22 tests + 3 driver tests (25 with `--include-ignored` on Mesa 25.2.8);
  clippy, fmt, MSRV 1.85, shellcheck, actionlint clean; `build-android.sh` with NDK r27d, three
  ABIs, 16 KB alignment, `NEEDED` only libc and libdl.

### Findings

Resolved: F-01, F-06 (pending merge of App #6), F-11, F-13, F-15, F-17, leads 1, 3, 4, 8.
Partly: F-07 (SDK exists, no consumer; graphics contract not mirrored). Open: F-02, F-03, F-04,
F-05, F-08, F-09, F-10, F-12, F-14, F-16.

New observation: Core answers `not_found` for every ztune failure, so the app separates "no ztune"
from "ztune failed" by Core's message prefix. A distinct Core error code would remove that
dependency (proposed, not done).

### Contracts and locks still to update

- After Modules #2 merges: re-pin the App to Core `345fc7f`, FAS `1c453db` and the Modules merge
  commit, and re-run full integration.
- SDK: add `zairenkai.graphics` schema 1 with captured zgfx output; re-copy App fixtures
  (`core/tuning*.json`) into `fixtures/captured/`.
- App: package `libzgfx.so` and a `graphics.*` subsystem once Graphics-Platform #1 is merged.

### Next three actions

1. Re-run the release gate on the CI full-integration APK and record the output (F-16).
2. SDK mirror of `zairenkai.graphics` and the new tuning fixtures (F-07).
3. App integration of zgfx after Graphics-Platform #1 merges.

### Maintainer actions

- Review and merge, when satisfied: Modules #2, App #6, Graphics-Platform #1, this Docs PR (F-04).
- Confirm Apache-2.0 for Zairenkai-Sdk, Zairenkai-Intelligence-Engine and
  Zairenkai-Graphics-Platform.
- Create a `dev` branch in Zairenkai-Tuning so its draft PR can be opened (F-05).
- Decide Tuning ADR 0001: keep the gated executor in ztune, or move writes to Core (F-09).
- Decide whether Core should give "ztune not installed" its own error code.
- Delete the unused `ZAIRENKAI_CI_APP_ID` / `ZAIRENKAI_CI_APP_PRIVATE_KEY` and uninstall the CI App (F-12).
- Allow this agent access to `ZairenkaiPrjctX/.github`, or run the foundation validator (F-08).
- Run `release.yml` from `dev` once the prerequisites hold (F-03).
- Choose devices for the first physical validation (Core, FAS, module lifecycle, zgfx) and for
  the Local AI runtime comparison (F-02).
- Install the plugins listed in `audit/ECOSYSTEM_AUDIT.md` §6 in an interactive session.
