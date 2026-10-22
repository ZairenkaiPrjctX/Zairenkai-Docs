# Ecosystem architecture

How the Zairenkai repositories fit together **today** (2026-10-22), and the target they move
towards. "Today" is what source at the named commits does; "target" is the plan in
`roadmap/IMPLEMENTATION_ROADMAP.md`. Ownership per capability: `REPOSITORY_RESPONSIBILITY_MATRIX.md`.

## 1. Runtime picture (today)

```text
                       ┌───────────────────────────── Android device (unverified) ───────────────────────────┐
                       │                                                                                       │
  user ──► Zairenkai-App (untrusted_app, Apache-2.0)                                                           │
           │  typed clients, consent, UI                                                                       │
           │                                                                                                   │
           ├── runs as its own uid ──► zart inspect (lib/<abi>/libzart.so)        ── Zairenkai-ART             │
           ├── runs as its own uid ──► zthermal snapshot (lib/<abi>/libzthermal.so) ── Zairenkai-ThermalTools │
           ├── reads asset ─────────► AFID device candidates (unverified)          ── Zairenkai-Database-Source│
           │                                                                                                   │
           └── su -c <fixed argv> (after the user enables root) ──────────────────────────────┐               │
                                                                                              ▼               │
                 root manager (Magisk / KernelSU / APatch) ── installs ──► module `zairenkai` (Zairenkai-Modules)
                                                                              │                               │
                                       zcored ctl (protocol v1) ◄─────────────┤ bin/zcored  Zairenkai-Core    │
                                         │  policy, arbitration, audit        │                               │
                                         ├── FAS socket ──────────────────────┤ bin/zfasd   Zairenkai-FAS     │
                                         │                                     │   writes scaling_min_freq only│
                                         ├── execve ztune capabilities|telemetry|apply                         │
                                         │                                     │ bin/ztune   Zairenkai-Tuning  │
                                         │                                     │   GPL-3.0, writes only with a │
                                         │                                     │   Tier D profile (none ships) │
                                         ├── execve zthermal snapshot (read only)                              │
                                         └── cmd power set-mode                                                │
                       └───────────────────────────────────────────────────────────────────────────────────────┘
```

Facts behind the picture (class S unless noted):

- App → Core: `su -c` with a fixed argv to `/data/adb/modules/zairenkai/bin/zcored ctl`
  (App `integration-core/.../core/Core.kt`). Core's socket is root-only (Core ADR-0005).
- App → FAS directly: `zfasd ctl` only when Core is down and the user confirms (App `Fas.kt`).
- Core writes no `/sys`, `/proc` or properties; FAS owns cpufreq floors, ztune owns the I/O
  scheduler and GPU governor nodes (Core `crates/zcore-platform/src/{fas,power,tuning,exec}.rs`).
- The module is assembled from components built in their own repositories at locked commits
  (Modules `components.lock.json`, `scripts/assemble.py`); ztune ships with its Corresponding
  Source (C: zip of 26 entries).
- The app installs the module only through the root manager's own installer, after
  confirmation, from the zip packaged in the APK, and only if the catalog is signed by a key in
  App `integration/module-trust.json`.

## 2. Build and supply chain (today)

```text
 Core ─┐                       ┌─► Modules assemble.py ─► zairenkai-vX.zip ─┐
 FAS  ─┼─ scripts/build-android.sh (NDK r27d, 16 KB LOAD check) ─┘          │
 Tuning┘     (+ GPL source archive)                                         ▼
 ART ────── scripts/build-android.sh ──► zart (3 ABIs) ──────────► App build-components.sh
 ThermalTools ─ scripts/build-android.sh ─► zthermal (3 ABIs) ───►   ├─ module_asset.py → catalog
 Database-Source ─ device candidates asset ──────────────────────►   ├─ sign-module-catalog.sh (release only)
                                                                      └─ Gradle → APK
                                                                             │
                                  verify_integration.py lock|access|build|apk|report [--release]
```

- App `integration/repository-lock.json` pins every source by full SHA; `build-components.sh`
  fetches each commit, and `verify_integration.py build` checks every artifact digest.
- Since App `e0d53bb` and Modules `298311c` the CI fetches these public repositories without
  credentials. Only `release.yml` uses secrets, all in the `release` environment.

## 3. Target additions

| Addition | Repository | Shape | Gate before it is used |
|---|---|---|---|
| Canonical contracts | Zairenkai-Sdk | JSON Schemas + compatibility tests against captured outputs; producers keep generating, SDK validates | ADR on canonical location (Phase 2) |
| Module catalog authority | Zairenkai-Modules | catalog schema and generation move from App to Modules; App keeps the trust anchor | old and new catalogs byte-compared (Phase 2) |
| Exact-device profiles | Zairenkai-Tuning | data (`profiles/`), resolver, tiers U/G/S/D/X | one hardware-tested Tier D profile; ADR 0001 decided |
| Graphics diagnostics | Zairenkai-Graphics-Platform | read-only Vulkan and GLES/EGL probes as separate subsystems | ADR on stack; no controls |
| Local AI | Zairenkai-Intelligence-Engine | on-device inference only, closed typed `ToolRegistry`, `IntentPlan` → Core | spike: offline model load, one typed tool call, airplane-mode proof |
| Entitlements | Zairenkai-License-Server | optional; local diagnostics never depend on it | scope ADR |
| Release orchestration | Zairenkai-Release | collects verified artifacts, SBOM, notices, checksums | must not bypass App's gate |
| Docs and site | Zairenkai-Docs, ZairenkaiPrjctX.github.io | this repository; static site with truthful status | link and accessibility checks |

## 4. Authority rules

1. Every privileged change goes through Core while Core answers. The app never runs a shell
   string; root commands are typed argv (`ShellScript` of `ShellWord`s).
2. Writes belong to one owner each: cpufreq floors → FAS, I/O scheduler and GPU governor → ztune
   (Tier D only), power mode → Core (`cmd power`). ThermalTools never writes.
3. A family or SoC match never authorizes a write. Unknown is reported as unknown.
4. Intelligence output is advice. A recommendation becomes an action only as a typed Core
   request, through policy, with user confirmation where policy requires it.
5. GPL-3.0 code (Tuning and its upstreams) never enters an Apache-2.0 repository; the boundary is
   a separate process and a documented JSON contract.
6. Trust anchors (module-trust list) are reviewed in Git; private keys stay outside Git, APK and
   logs.
