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
- [ ] Upstream catalog audited for graphics and local AI (F-15; done for Tuning)

Acceptance: findings carry severity and evidence class; no device claim without hardware.

## Phase 1 · Unblock reliable integration

- [x] App CI and release fetch public locked SHAs without a token (App `e0d53bb`)
- [x] Modules CI likewise (Modules `298311c`)
- [x] App inventory reconciled: 17 repositories, Tuning added, anime-extensions excluded
- [x] Stale docs fixed in App, Core, FAS, ART, ThermalTools, Database-Source
- [x] Release doc reconciled with the trust list (App `d0db6b7`)
- [ ] **App full-integration job green on GitHub** without `allowMissingComponents` (run
      39827139485 is the first attempt)
- [ ] Modules `module` job green on GitHub
- [ ] Release gate re-run on the full-integration APK; output recorded
- [ ] Central `.github` inventory updated (needs access)

Acceptance: a CI run id with every full-integration step green, and the integration report.

## Phase 2 · Contracts and module authority

- [ ] ADR in Zairenkai-Sdk: producers canonical, SDK mirrors and validates (`architecture/CONTRACTS_AND_VERSIONING.md` §2)
- [ ] SDK schema mirrors for Core v1, FAS, zthermal, zart, AFID, tuning; tests against captured fixtures
- [ ] SDK-defined schemas: module catalog, Local-AI types, entitlements
- [ ] Catalog generation moved from App to Modules, with old/new catalogs compared byte for byte
- [ ] App re-pinned to Modules with ztune and to Core with the tuning adapter (F-06); typed tuning client

Acceptance: SDK CI validates every captured fixture; App full-integration green at the new pins.

## Phase 3 · Empty repositories, one at a time

- [ ] Intelligence-Engine spike: ADR LiteRT-LM vs llama.cpp (licences, ABI, size), one offline
      model load on a host with no network, one typed tool call through a closed `ToolRegistry`,
      airplane-mode proof; no remote AI API
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

## Session checkpoint · 2026-10-22

**Phase:** 1 (Phase 0 documents written). **Last completed task:** these six documents.

### Changes in this session (branch `ccr-63f8aa87-1b0paj` in every repository)

| Repository | Commits | PR |
|---|---|---|
| Zairenkai-Tuning | `ab09e36`, `450a624`, `0c226b2` (GPL-3.0 `ztune`, SoC registry, profiles/tiers) | none possible: no default branch |
| Zairenkai-Core | `6fd0f79`, `8931cbb` (tuning adapter, read-only unless authorized), `dec9d79` (docs) | draft #3 |
| Zairenkai-Modules | `d7f435b`, `bd1eb06` (ztune + Corresponding Source in the module), `298311c` (CI without App) | draft #1 |
| Zairenkai-App | `e0d53bb` (CI/release without App, inventory), `d0db6b7` (release doc) | draft #5 |
| Zairenkai-FAS | `6f9bccc` (docs) | draft #4 |
| Zairenkai-ART | `a999e98` (docs), branch restarted from `dev` `0d8cd72` | none yet |
| Zairenkai-ThermalTools | `6ed9a30` (docs), from `dev` `3bc71fe` | none yet |
| Zairenkai-Database-Source | `52fa33c` (docs), from `dev` `d9c4290` | none yet |
| Zairenkai-Docs | this change | draft (this branch) |

All commits are SSH-signed (`gpgsig` present on each commit object).

### Commands and results

See `audit/ECOSYSTEM_AUDIT.md` §2. Summary: FAS 82, ThermalTools 109, ART 26 + 16, AFID 325,
Core 74, Tuning 68, Modules 17, App verifier 17: all passed on hosts. App Gradle: **blocked**
(Maven Central HTTP 429). App CI full-integration: first run without a token in progress
(39827139485).

### Findings

Fixed: F-11 (stale facts), lead 1 (App inventory), lead 3 (CI token), lead 4 (release doc).
Open: F-01 (pending CI run), F-02, F-03, F-04, F-05, F-06, F-07, F-08, F-09, F-10, F-12,
F-13, F-14, F-15, F-16.

### Contracts and locks still to update

- App lock: Modules `ea138ef` → a Modules commit with ztune; Core `4cdfe8e` → a commit with the
  tuning adapter. Only after full-integration passes at the new pins.
- Modules lock: Core `8931cbb` (now `dec9d79`, docs only: no re-pin needed), FAS `99cfb18`
  (now `6f9bccc`, docs only).

### Next three actions

1. Read run 39827139485; fix any full-integration failure until the job is green (F-01).
2. Re-pin App to Modules with ztune and add the typed tuning client and fixtures (F-06).
3. Zairenkai-Sdk Phase 2: ADR and schema mirrors validated against the captured fixtures (F-07).

### Maintainer actions

- Create a `dev` branch in Zairenkai-Tuning so its draft PR can be opened (F-05).
- Decide Tuning ADR 0001: keep the gated executor in ztune, or move writes to Core (F-09).
- Review and merge Core #3, FAS #4, Modules #1, App #5 when CI is green (F-04).
- Delete the unused `ZAIRENKAI_CI_APP_ID` / `ZAIRENKAI_CI_APP_PRIVATE_KEY` and uninstall the CI App (F-12).
- Allow this agent access to `ZairenkaiPrjctX/.github`, or run the foundation validator (F-08).
- Run `release.yml` from `dev` once the gate's prerequisites hold (F-03).
- Choose the device for the first physical validation (F-02).
- Install the plugins listed in `audit/ECOSYSTEM_AUDIT.md` §6 in an interactive session.
