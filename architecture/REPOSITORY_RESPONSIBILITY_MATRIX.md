# Repository responsibility matrix

One owner per capability. "Contract" is the interface consumers depend on; "Privilege" is what
the code runs as on a device; "Tests" are the checks that exist today (2026-10-22). Status words:
**implemented** (code and tests exist), **topic branch** (not on the default branch), **empty**
(no code), **unverified on device** applies to every implemented row.

| Capability | Owner | Consumers | Contract | Privilege | Tests today | Status |
|---|---|---|---|---|---|---|
| Android UI, consent, typed clients | Zairenkai-App | users | APK; `integration/*.json` | `untrusted_app`; root only through the user's root manager | `:integration-core:test`, `:app:testDebugUnitTest` (Robolectric), lint, verifier tests | implemented; App PR #5 |
| Control plane: policy, arbitration, audit, module inventory | Zairenkai-Core | App, Intelligence (future) | `zcored ctl` protocol v1 JSON (`docs/api/PROTOCOL.md`, `zcore-domain/src/protocol.rs`) | root | `cargo test` 74, fixtures, end-to-end socket test | implemented; PR #3 |
| CPU frequency floors from frame telemetry | Zairenkai-FAS | Core (adapter), App (fallback) | control socket JSON (`docs/INTEGRATION.md`) | root; `scaling_min_freq` only | `cargo test` 82 | implemented; PR #4 |
| Thermal diagnostics, vendor config inspection | Zairenkai-ThermalTools | Core (adapter), App | `zthermal snapshot` schema 1 (`schemas/`) | app uid; root for `--framework` | `cargo test` 109, schema checks | implemented (`dev`) |
| ART inspection; upstream ART patch decisions | Zairenkai-ART | App | `zart inspect` / `dexopt-status` JSON `schema: 1` | app uid; root for `dexopt-status` | `cargo test` 26, `tools/tests` 16 | implemented (`dev`) |
| Firmware metadata catalog (AFID) | Zairenkai-Database-Source | App, Core (device lookup) | `schemas/*.schema.json`; device-candidates asset | none on device | `pytest` 325, mypy, ruff, policy | implemented (`dev`); 0 verified devices |
| Single root module packaging | Zairenkai-Modules | App (installer), root managers | `components.lock.json`, `component.json`, module zip layout | installer scripts run by the root manager | unittest 17, shellcheck | implemented; PR #1 |
| Module catalog and signature | Zairenkai-App today → Zairenkai-Modules (target) | App | `integration/system-modules.json`, `assets/system-modules.json(.sig)` | — | `SystemModuleTrustTest`, verifier | in App; move planned (Phase 2) |
| Module trust anchor | Zairenkai-App | App | `integration/module-trust.json` | — | `SystemModuleTrustTest` | implemented; 1 key |
| Telemetry, SoC identity, exact-device tuning profiles | Zairenkai-Tuning (GPL-3.0) | Core | `zairenkai.tuning` schema 1 (`docs/CONTRACT.md`) | root; writes only under Tier D grant | `cargo test` 68 | topic branch only; no default branch |
| Canonical cross-repository contracts | Zairenkai-Sdk | all | — | — | — | empty |
| Graphics diagnostics (Vulkan, GLES/EGL) | Zairenkai-Graphics-Platform | Core, App | — | read-only target | — | empty |
| On-device AI, intent planning, typed tool routing | Zairenkai-Intelligence-Engine | App (host), Core (authority) | planned: `IntentPlan`, `ToolInvocation`, `ActionPreview`, `UserApproval`, `ExecutionResult`, `AgentAuditEvent`, `ModelManifest` | app process; no root | — | empty |
| Entitlements | Zairenkai-License-Server | App, Release | — | server (optional) | — | empty |
| Release orchestration, SBOM, notices | Zairenkai-Release (target); App `release.yml` today | maintainers | — | CI with protected secrets | App `verify_integration.py report --release` | App only; Release empty |
| Architecture, ADRs, audit, roadmap | Zairenkai-Docs | everyone | these documents | — | — | started (this change) |
| Public site | ZairenkaiPrjctX.github.io | public | — | — | — | empty |
| Agent foundation, org inventory, validator | .github | all repositories | `agent-foundation/` | — | validator tests (not run: access) | not inspected |

## Write ownership on a device

| Node or service | Only writer | Bounds |
|---|---|---|
| `cpufreq/policy*/scaling_min_freq` | zfasd | within `[cpuinfo_min, current scaling_max]`, restore-before-write, lease, read-back |
| `queue/scheduler` of block devices | ztune | values the device advertises and a Tier D profile grants; journal, read-back, rollback |
| GPU devfreq `governor` | ztune | same as above |
| Power save mode | Core (`cmd power set-mode`) | policy, read-back |
| Thermal nodes, trip points, cooling devices | nobody | ThermalTools `policy::denied_target`, Tuning and FAS rules |
| `/data/adb/modules/zairenkai` | the root manager's installer | App never writes it |
