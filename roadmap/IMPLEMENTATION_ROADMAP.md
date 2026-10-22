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
- [ ] Typed tuning client in the app (F-06 remainder)

Acceptance: SDK CI validates every captured fixture; App full-integration green at the new pins.

## Phase 3 · Empty repositories, one at a time

- [x] Intelligence-Engine spike (IE PR #1): ADR LiteRT-LM vs llama.cpp, one offline model load,
      typed tool calls through a closed `ToolRegistry`, airplane-mode proof on a Linux host; no
      remote AI API. Eval 9/16: the 0.5B model is too weak alone
- [ ] Intelligence-Engine Phase B: runtime comparison on Android devices (needs devices)
- [ ] Graphics-Platform: ADR on stack; read-only Vulkan probe and GLES/EGL probe as separate subsystems
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

## Session checkpoint · 2026-10-22 (end of session)

**Phase:** 2 and 3 started (Phase 1 done except the central `.github` inventory and the release
gate re-run). **Last completed task:** Local AI Phase A spike (IE PR #1) and PR descriptions.

### Changes in this session (branch `ccr-63f8aa87-1b0paj` unless noted)

| Repository | Commits | PR | CI |
|---|---|---|---|
| Zairenkai-Tuning | `ab09e36`, `450a624`, `0c226b2` | none possible: no default branch | built by Modules and App CI |
| Zairenkai-Core | `6fd0f79`, `8931cbb`, `dec9d79` | draft #3 | green (39827498175) |
| Zairenkai-Modules | `d7f435b`, `bd1eb06`, `298311c` | draft #1 | green, `module` job included (39827725469) |
| Zairenkai-App | `e0d53bb`, `d0db6b7`, `e1ca387` | draft #5 | green, full-integration included (39827139485, 39830066001) |
| Zairenkai-FAS | `6f9bccc` | draft #4 | green (39827237501) |
| Zairenkai-ART | `a999e98` | draft #3 | green (39828342234) |
| Zairenkai-ThermalTools | `6ed9a30` | draft #3 | green (39828355708) |
| Zairenkai-Database-Source | `52fa33c`, `10a28fa` (port of #14) | draft #13; fix draft #14 (`ccr-63f8aa87-1b0paj-certifi`) | #13 `audit` was red on certifi; re-run pending after the port |
| Zairenkai-Sdk | `41481ec` | draft #1 | green (39831321147) |
| Zairenkai-Intelligence-Engine | `0e56b90` | draft #1 | **no workflow run started** (Actions may be disabled in this repository) |
| Zairenkai-Docs | `7735d1f` + this update | draft #1 | no CI |

Every commit listed carries an SSH signature (`gpgsig` on the commit object).

### Commands and results

`audit/ECOSYSTEM_AUDIT.md` §2. Highlights: App full-integration green twice in CI; Modules
`module` job green; SDK 25 tests and 33 fixtures; IE 14 tests, real-model test, offline proof
(0 network attempts), eval 9/16. App Gradle locally: blocked (Maven Central HTTP 429); CI is the
evidence.

### Findings

Resolved: F-01, F-11, F-13, leads 1, 3, 4, 8. Partly: F-06 (typed tuning client open), F-07
(SDK not consumed yet), F-15 (graphics open). Open: F-02, F-03, F-04, F-05, F-08, F-09, F-10,
F-12, F-14, F-16, F-17 (fix in #14).

### Contracts and locks still to update

- After the maintainer merges Core #3, FAS #4, Modules #1: re-pin App and Modules to the merge
  commits and re-run full integration.
- SDK `fixtures/captured/` re-copied whenever App or Core re-capture.
- IE `contracts/sdk/` follows the SDK pin (`41481ec`).

### Next three actions

1. Typed tuning client in the app over Core's `tuning` method, with captured fixtures (F-06).
2. Re-run the release gate on the CI full-integration APK and record the output (F-16).
3. Graphics-Platform: stack ADR and a read-only Vulkan / GLES probe (Phase 3).

### Maintainer actions

- Create a `dev` branch in Zairenkai-Tuning so its draft PR can be opened (F-05).
- Decide Tuning ADR 0001: keep the gated executor in ztune, or move writes to Core (F-09).
- Review and merge, when satisfied: Core #3, FAS #4, Modules #1, App #5, ART #3, ThermalTools #3,
  Database-Source #14 then #13, Sdk #1, Intelligence-Engine #1, Docs #1 (F-04).
- Confirm Apache-2.0 for Zairenkai-Sdk and Zairenkai-Intelligence-Engine.
- Check that GitHub Actions is enabled in Zairenkai-Intelligence-Engine.
- Delete the unused `ZAIRENKAI_CI_APP_ID` / `ZAIRENKAI_CI_APP_PRIVATE_KEY` and uninstall the CI App (F-12).
- Allow this agent access to `ZairenkaiPrjctX/.github`, or run the foundation validator (F-08).
- Run `release.yml` from `dev` once the prerequisites hold (F-03).
- Choose devices for the first physical validation and for the Local AI runtime comparison (F-02).
- Install the plugins listed in `audit/ECOSYSTEM_AUDIT.md` §6 in an interactive session.
