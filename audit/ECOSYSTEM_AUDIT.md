# Ecosystem audit

Scope: the 16 Zairenkai repositories of the `ZairenkaiPrjctX` organization. `anime-extensions`
is excluded by the maintainer and was not cloned, built or read for this audit.

Audit date: 2026-10-22. Each finding names its evidence class:

| Class | Meaning |
|---|---|
| **S** | Verified by source inspection at the named commit |
| **C** | Verified by a command run in this session; its result is quoted |
| **CI** | Verified by a GitHub Actions run (id given) |
| **H** | Verified on a host only (x86_64 or aarch64 Linux, no Android device) |
| **D** | Verified on real Android hardware. **No finding has this class**: no device was available |
| **N** | Not verified, or blocked (the blocker is named) |

Nothing in this document claims that a component works on a device, supports all devices, or is
release-ready.

## 1. Repositories and heads

Heads from `git ls-remote --heads` and `list_repos`, 2026-10-22. Every repository is **public**.
"Code" counts tracked files outside the agent foundation (`AGENT/`, `.claude/`, `AGENTS.md`,
`CLAUDE.md`, `.gitignore`) on the default branch.

| Repository | Default branch head | Topic branch (`ccr-63f8aa87-1b0paj`) | Code | State |
|---|---|---|---|---|
| Zairenkai-App | `dev` `c7b85e8` | `e1ca387` (draft PR #5) | 145 files | Android app, integration layer, scripts |
| Zairenkai-Core | `dev` `11e9742` | `dec9d79` (draft PR #3) | 75 | Rust control plane |
| Zairenkai-FAS | `dev` `be08b2d` | `6f9bccc` (draft PR #4) | 50 | Rust frame-aware daemon |
| Zairenkai-ThermalTools | `dev` `3bc71fe` | `6ed9a30` (new, docs only) | 132 | Rust read-only thermal CLI |
| Zairenkai-ART | `dev` `0d8cd72` | `a999e98` (new, docs only) | 35 | Rust `zart`, Python upstream tool |
| Zairenkai-Database-Source | `dev` `d9c4290` | `10a28fa` (draft PR #13; fix PR #14) | 898 | AFID Python pipeline and catalog |
| Zairenkai-Modules | `dev` `f226502` | `298311c` (draft PR #1) | 19 on the topic branch | Single `zairenkai` module assembly |
| Zairenkai-Tuning | **none** | `0c226b2` (only branch) | 70 | GPL-3.0 `ztune` |
| Zairenkai-Sdk | `dev` `d482afe` | `41481ec` (draft PR #1) | 0 on `dev` | contracts on the topic branch |
| Zairenkai-Intelligence-Engine | `dev` `0191e36` | `0e56b90` (draft PR #1) | 0 on `dev` | Local AI spike on the topic branch |
| Zairenkai-Graphics-Platform | `dev` `b3dcca4` | – | 0 | foundation only |
| Zairenkai-License-Server | `dev` `d563033` | – | 0 | foundation only |
| Zairenkai-Release | `dev` `0d80427` | – | 0 | foundation only |
| Zairenkai-Docs | `dev` `068bf87` | this change | 0 before this change | documentation |
| ZairenkaiPrjctX.github.io | `dev` `27362a4` | – | 0 | foundation only |
| .github | `dev` `4cdab7f` (from the preliminary audit) | – | – | **not inspected**: see F-08 |

Evidence: C (`git ls-remote`, `git ls-tree -r origin/dev`), 2026-10-22.

## 2. Checks executed in this session

| Repository @ commit | Command | Result | Class |
|---|---|---|---|
| FAS @ `99cfb18` | `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test --workspace` | fmt ok, clippy clean, 82 passed, 0 failed | C/H |
| ThermalTools @ `3bc71fe` | fmt, clippy, `cargo test --workspace`, `scripts/check-schemas.py` | 109 passed; 4 snapshots, 7 inspections, 1 profile valid | C/H |
| ART @ `0d8cd72` | fmt, clippy, `cargo test`, `python3 -m unittest discover -s tools/tests`, `art_upstream.py validate` | 26 + 16 passed; `ok: 5 sources, releases ['android-17']` | C/H |
| Database-Source @ `d9c4290` | `pytest -q`, ruff check/format, `mypy` strict, `afid policy check`, `afid security scan` | 325 passed; ruff clean; mypy no issues in 56 files; policy passed; no findings | C/H |
| Core @ `8931cbb` | fmt, clippy, `cargo test --workspace`, MSRV 1.85, NDK build | 74 passed, clean, component built | C/H |
| Tuning @ `0c226b2` | fmt, clippy, `cargo test --workspace`, MSRV 1.85, shellcheck, NDK build, offline rebuild from the source archive | 68 passed (44 + 13 + 11), clean, exit 0 | C/H |
| Modules @ `bd1eb06` / `298311c` | `python3 -m unittest discover -s tests`; shellcheck; actionlint; `assemble.py` on real arm64 components | 17 passed; clean; `zairenkai-v0.2.0.zip`, 26 entries, SHA-256 `f4ad8d64…400d` | C/H |
| App @ `d0db6b7` | `verify_integration.py lock`, `access`; verifier tests; shellcheck; actionlint | lock ok; access ok without credentials; 17 passed; clean | C/H |
| App @ `d0db6b7` | `./gradlew :integration-core:test` | **blocked**: Maven Central answered HTTP 429 to this session before compilation | N |
| App, CI | run 39706970377 (`dc2e985`) | scripts ✓, reduced ✓, full-integration ✗ at "Read-only token for the component repositories" | CI |
| App, CI | run 39827139485 (`d0db6b7`, first run without the App token) | **all jobs green, full-integration included**: the first CI build of the integrated APK with native components | CI |
| App, CI | run 39830066001 (`e1ca387`, module with ztune, Core `8931cbb`) | all jobs green | CI |
| Modules, CI | run 39827725469 (`298311c`) | `module` job green for the first time (Core + FAS + Tuning built and assembled) | CI |
| Sdk @ `41481ec` | unittest; `zsdk.py validate`; `check-mirrors`; CI run 39831321147 | 25 passed; 33/33 fixtures valid; mirrors ok; CI green | C/CI |
| Intelligence-Engine @ `0e56b90` | unittest; real-model test; `tools/offline-proof.sh`; `tools/eval.py` | 14 passed + 1 real-model test passed; offline proof exit 0, 0 network attempts; eval 9/16 | C/H |
| Database-Source PR #13 / #14 | CI `audit` (pip-audit) | failed on certifi 2026.10.18 (PYSEC-2026-4320) on every branch; fixed in #14 (`ccr-…-certifi`), ported to #13 as `10a28fa` | CI/C |

Logs: the session scratchpad (`checks/*.log`); not committed.

## 3. Audit leads re-verified

The preliminary remote audit listed eight leads. Each was reproduced against current clones.

| Lead | Result | Class |
|---|---|---|
| 1. App inventory stale (16, no Tuning, old visibility) | **Confirmed and fixed** in App `e0d53bb`: 17 repositories, all public, Tuning added, anime-extensions marked excluded. The central `.github` inventory is not reconciled (F-08) | C |
| 2. App lock `merged: false` stale | **Lead itself stale.** The current topic-branch lock pins ART `0d8cd72`, ThermalTools `3bc71fe`, Database-Source `d9c4290` (`merged: true`, ancestors of `dev`) and Core `4cdfe8e`, FAS `99cfb18`, Modules `ea138ef` with `merged: false`, which is **correct**: none of the three is an ancestor of its `dev` (`git merge-base --is-ancestor`). The SHAs named in the lead (`34e93e3`, `333365e`, `8c6cea2`, `9d4b310`) are no longer pinned | C |
| 3. CI requires a GitHub App for public sources | **Confirmed and fixed** in App `e0d53bb` (ci.yml, release.yml, verifier, docs, setup script) and Modules `298311c` | C |
| 4. Release docs say the trust list is empty | **Confirmed and fixed** in App `d0db6b7`: the trust list holds `4330305a…`; the last local gate run failed on two blockers (APK and catalog unsigned) | S/C |
| 5. Eight repositories foundation-only; Tuning empty | Sdk, Intelligence-Engine, Graphics-Platform, License-Server, Release, Docs and the website: **confirmed** (0 code files). Modules: **no longer true** (topic branch). Tuning: **no longer true** (topic branch `0c226b2`, no default branch) | C |
| 6. AFID metadata-only, sources disabled | **Confirmed**: 0 verified devices, 354 unverified staged candidates (`database/statistics.json:213`), registry sources disabled | S/C |
| 7. ThermalTools read-only, FAS bounded, zart never on hardware | **Confirmed** by source and tests; no device validation anywhere (F-02) | S/C |
| 8. Stale "no app exists" and visibility claims | **Confirmed and fixed**: FAS `6f9bccc`, ART `a999e98`, ThermalTools `6ed9a30`, Database-Source `52fa33c`, Core `dec9d79`, App `e0d53bb` | C |

## 4. Findings

### BLOCKER

**F-01 · App full-integration build had never passed on GitHub. RESOLVED** (run 39827139485, then 39830066001).
`Zairenkai-App/.github/workflows/ci.yml` job `full-integration`. Until `dc2e985` it stopped at the
GitHub App token step (run 39706970377), so no CI run has built the native components into the
APK. Impact: no CI evidence that the integrated app builds. Fix: token removed (`e0d53bb`); the
first run without it is 39827139485. Class: CI (failure), fix N until that run completes.

**F-02 · Nothing has run on an Android device.**
Every component: App, zcored, zfasd, zthermal, zart, ztune, the `zairenkai` module scripts.
Impact: install, SELinux, root prompts, sysfs layouts and read-back behaviour are unknown.
Fix: the device procedures (`Zairenkai-App/docs/testing/device-validation.md`) on a physical
device chosen by the maintainer. Class: N (no hardware).

**F-03 · No signed APK or signed module catalog exists; `release.yml` has never run.**
`Zairenkai-App/.github/workflows/release.yml`. The gate fails on "APK unsigned" and "catalog
unsigned". The five signing secrets are reported set by name only. Fix: a maintainer-dispatched
`release.yml` run from `dev`. Class: S/C (gate run 2026-10-21), N for the secrets' content.

### HIGH

**F-04 · The app pins three components on unmerged topic branches.**
`Zairenkai-App/integration/repository-lock.json:60,72,84` (`merged: false` for Core `4cdfe8e`,
FAS `99cfb18`, Modules `ea138ef`). Impact: a release would ship code that is not on any default
branch. Fix: the maintainer reviews and merges Core PR #3, FAS PR #4 and Modules PR #1, then the
lock is re-pinned to the merge commits after a full-integration build. Class: C.

**F-05 · Zairenkai-Tuning has no default branch.**
`git ls-remote` shows only `ccr-63f8aa87-1b0paj`. Impact: no pull request can be opened, and
Modules' lock pins a commit reachable only from a topic branch. Fix: the maintainer creates `dev`
(from an empty commit or from `0c226b2`) so a draft PR can be opened. Class: C.

**F-06 · The app did not integrate Zairenkai-Tuning. RESOLVED** in App `e1ca387` (module with ztune pinned) and App PR #6 (`CoreClient.tuning()` over Core's `tuning` method, fixtures captured from Core's adapter running the real ztune, record `tuning.telemetry`; CI run 39888821755 green), not merged yet.
App lock pins Modules `ea138ef` (Core + FAS only); Modules `bd1eb06`+ adds ztune and its
Corresponding Source. The app has no tuning client and no captured `zairenkai.tuning` fixtures.
Fix: re-pin Modules and Core once their PRs settle, add a typed client parsing the captured
documents (Core `crates/zcore-platform/tests/fixtures/tuning/`). Class: S.

**F-07 · No canonical contract set. ADDRESSED** by Zairenkai-Sdk PR #1 (merged as `ed44c90`); no consumer validates against it yet, and the new `zairenkai.graphics` contract is not mirrored yet.
Contracts live with their producers: Core `crates/zcore-domain/src/protocol.rs`, ThermalTools
`schemas/`, AFID `schemas/`, Tuning `docs/CONTRACT.md`, App `contracts/` fixtures,
`integration/system-modules.json`. Zairenkai-Sdk is empty. Impact: no single compatibility
check across repositories. Fix: Phase 2 (`architecture/CONTRACTS_AND_VERSIONING.md`). Class: S.

**F-08 · The central `.github` inventory, generator and validator were not audited.**
Attaching `ZairenkaiPrjctX/.github` was refused by this session's permission policy, so its
`agent-foundation/inventory/repositories.json` is not reconciled and the foundation validator
(`validate_agent_setup.py`) was not run in any repository. Fix: the maintainer allows access, or
runs the validator and the inventory update. Class: N (permission).

**F-09 · Tuning's executor role conflicts with the target architecture.**
`ZAIRENKAI_CROSS_DEVICE_GRAPHICS_TUNING_ARCHITECTURE.md:33` says Tuning is "never a root
executor"; `ztune apply` (`Zairenkai-Tuning/crates/ztune/src/apply.rs:92`) writes
`io.scheduler` and `gpu.governor`, and only under a Tier D `device_validated` profile grant. No
such profile ships (`profiles/index.json` is `[]`), so no write is possible on any device today.
Fix: maintainer decision recorded in Tuning `docs/adr/0001-write-path-tiers.md` (keep the gated
executor, move writes into a Core adapter, or drop them). Class: S/C (tests
`without_a_device_validated_profile_nothing_is_written`).

### MEDIUM

**F-10 · App Gradle checks could not run in this session.** Maven Central returned HTTP 429 to
`:integration-core:compileKotlin` dependency resolution. CI is the reference. Class: N
(environment).

**F-11 · Stale facts across repositories.** Visibility "private", "Zairenkai-App has no code",
"353 candidates" where `statistics.json` says 354, Modules/App "foundation only" in Core.
Fixed: see lead 8. Class: C.

**F-12 · Unused CI App values stay on GitHub.** `ZAIRENKAI_CI_APP_ID` and
`ZAIRENKAI_CI_APP_PRIVATE_KEY` (App, repository level) are no longer read by any workflow.
Fix: the maintainer deletes them and uninstalls the App. Class: S.

**F-13 · Component CI on topic branches is not evidence for the app's pins. RESOLVED** for Core `8931cbb` through App and Modules CI (both build it at that commit). Core and FAS CI ran
green on their dispatches (39701977860 at `4cdfe8e`, 39701982927 at `99cfb18`), but Core's
lock in Modules is now `8931cbb`, whose component was built only locally. Fix: dispatch Core CI
at `8931cbb` (or the eventual merge commit). Class: CI/C.

**F-14 · AFID pull-request workflows returned `action_required` on `afid/updates`** (preliminary
audit). Not re-checked in this session. Class: N.

### LOW

**F-15 · Upstream reference catalog not yet audited for graphics and local AI. RESOLVED** for both: local AI in Intelligence-Engine ADR 0001 (llama.cpp `097f5b53` MIT, LiteRT-LM `bda62485` Apache-2.0, Gallery `8f5f2a0b` Apache-2.0, Qwen2.5-0.5B GGUF `9217f5db` Apache-2.0); graphics in Graphics-Platform PR #1 `docs/THIRD_PARTY.md` (table in §5). Perfetto and nowinandroid remain unaudited: nothing uses them yet. Class: N.

**F-17 · AFID security audit red on every branch** (certifi PYSEC-2026-4320). Fix: Database-Source PR #14. Class: CI/C.

**F-16 · Release gate output in `docs/testing/verification-report.md` predates the single-module
re-pin.** Re-run it after F-01 passes. Class: S.

## 5. External sources audited

| Subsystem | Source | Commit | Licence | Use | Decision |
|---|---|---|---|---|---|
| Tuning (telemetry) | Rve27/RvSystem-Monitor | `b07fe275` | GPL-3.0 | collectors adapted into `ztune-core` | adapted, headers and change lists kept (Tuning `docs/PROVENANCE.md`) |
| Tuning (I/O, GPU governor) | Zy0x/Lynx | `e56d6cd5` | GPL-3.0 | scheduler and governor handling | adapted; its `references/` device trees **rejected** as evidence (reconstructions, not vendor sources) |
| Tuning (GPU table) | libxzr/KonaBess | `ff387dc7` | GPL-3.0 | DTS GPU table parsing, chip labels | adapted; no flashing, no boot-image repack |
| Tuning (SoC registry) | torvalds/linux | `7be4bc52` | GPL-2.0 (facts only) | socinfo ids and DT compatibles extracted as data | 443 identifiers, generated by `scripts/update-soc-registry.py` |
| Tuning (Tensor, Qualcomm board names) | LineageOS device trees gs101 `750d40c5`, gs201 `f75ebcf4`, zuma `9eeeb27b`, garnet `5b3cb512` | as listed | Apache-2.0 | `ro.board.platform` values | data only |
| GPL boundary | — | — | — | Core and Modules (Apache-2.0) run ztune as a separate program and ship its source | no GPL code in Apache repositories (S: Core `tuning.rs` parses JSON only) |
| Graphics (Vulkan) | KhronosGroup/Vulkan-Tools | `a967170a` | Apache-2.0 | `vulkaninfo` as a reference for which properties matter | studied, not bundled: zgfx enumerates through the loader (Graphics-Platform ADR 0001) |
| Graphics (Vulkan) | KhronosGroup/Vulkan-Headers | `73814d00` | Apache-2.0 / MIT per file | none directly | the `ash` 0.38.0+1.3.281 crate (MIT OR Apache-2.0) is generated from vk.xml 1.3.281 |
| Graphics (conformance) | KhronosGroup/VK-GL-CTS | `d18e4c68` | Apache-2.0 | out of scope for a probe | not used |
| Graphics (GLES) | google/angle | `df00ff99` | BSD-3-Clause | reference only | never installed or selected |
| Graphics (loading) | nagisa/rust_libloading | `f5dc3b81` (crate 0.8.9) | ISC | `dlopen` of the system graphics libraries | dependency |
| Local AI | llama.cpp, LiteRT-LM, google-ai-edge/gallery, Qwen2.5-0.5B GGUF | see F-15 | MIT, Apache-2.0 | runtime spike | Intelligence-Engine ADR 0001 |
| App UI | nowinandroid, Perfetto | — | — | — | not audited: nothing uses them |

## 6. Plugins

The session cannot install Claude Code plugins itself. A human runs, in an interactive
Claude Code session:

```
/plugin marketplace add anthropics/claude-plugins-official
/plugin install frontend-design@claude-plugins-official
/plugin install feature-dev@claude-plugins-official
/plugin install code-review@claude-plugins-official
/plugin install security-guidance@claude-plugins-official
/plugin install code-simplifier@claude-plugins-official
```

Context7 and Playwright: take their exact IDs from `/plugin` (marketplace browser); they are not
guessed here. The repositories' own skills (`/repo-audit`, `/test-and-verify`,
`/security-review`, `/release-readiness`) were the audit method used.
