# Contracts and versioning

Every interface that crosses a repository boundary, where its definition lives today, and the
rules for changing it. Inventory from source at the commits in `audit/ECOSYSTEM_AUDIT.md`
(2026-10-22).

## 1. Inventory

| Contract | Producer (definition today) | Version field | Consumers | Captured fixtures |
|---|---|---|---|---|
| Core control protocol | Core `crates/zcore-domain/src/protocol.rs` (`PROTOCOL_VERSION = 1`), `docs/api/PROTOCOL.md` | `v: 1` in every request and response | App (`Core.kt`) | App `integration-core/src/test/resources/contracts/core/`, from the real zcored |
| FAS control socket and status | FAS `crates/zfasd`, `docs/INTEGRATION.md` ("Schema version 1") | `schema_version` | Core `fas.rs`, App `Fas.kt` | App `contracts/fas/` |
| FAS configuration | FAS `crates/zfas-core/src/config.rs` (`SCHEMA_VERSION = 1`, with migration) | `schema_version` | Modules `module/defaults/fas-config.toml` | — |
| ThermalTools snapshot, capability, inspection, profile | ThermalTools `schemas/*.schema.json` (4 files) | `schema_version: 1` | Core `thermaltools.rs`, App | App `contracts/zthermal/`; ThermalTools `scripts/check-schemas.py` |
| zart inspect / dexopt-status | ART `crates/zart/src/inspect.rs` (`schema: 1`) | `schema` | App `Zart.kt` | App `contracts/zart/` |
| AFID records and assets | Database-Source `schemas/*.schema.json` (11 files) | per schema | App asset, Core `afid.rs` | AFID `tests/schemas/` |
| Tuning document | Tuning `crates/ztune-core/src/contract.rs`, `docs/CONTRACT.md` (`zairenkai.tuning` schema 1) | `schema` + `kind` | Core `tuning.rs` | Core `crates/zcore-platform/tests/fixtures/tuning/` (captured from Tuning `0c226b2`) |
| Tuning profile | Tuning `crates/ztune-core/src/profile.rs` (`deny_unknown_fields`), `docs/PROFILES.md` | `schema_version` | ztune only | synthetic test profile, labelled `synthetic: true` |
| Component manifest | Core/FAS/Tuning `scripts/build-android.sh` → `component.json` | `format` | Modules `assemble.py` | Modules `tests/` (stand-ins) |
| Module lock | Modules `components.lock.json` | `format: 1` | Modules CI, App `build-components.sh` | — |
| Module catalog and signature | App `integration/system-modules.json` (`schema: 1`) → `assets/system-modules.json` + `.sig` | `schema` | App installer | App `contracts/system-modules/` from the real zip |
| Module trust list | App `integration/module-trust.json` | — | App | `SystemModuleTrustTest` |
| Source lock and inventory | App `integration/repository-lock.json` (`schema_version: 1`), `integration/organization-repositories.json` (`schema: 1`) | as named | App verifier, `Ecosystem.kt` | — |
| Local-AI contracts (`IntentPlan`, `ToolInvocation`, `ActionPreview`, `UserApproval`, `ExecutionResult`, `AgentAuditEvent`, `ModelManifest`, `DeviceCapabilitySnapshot`) | none yet | — | — | — |
| Licence/entitlement | none yet | — | — | — |

## 2. Canonical source: decision proposed for Phase 2

Each producer already owns a working definition, and several ship captured fixtures. Moving
every definition into Zairenkai-Sdk would be a rewrite of stable v1 interfaces. Proposal (to be
recorded as an ADR in Zairenkai-Sdk before code):

1. **The producer stays canonical** for its own output (Core protocol, FAS socket, zthermal,
   zart, AFID, ztune). Zairenkai-Sdk holds a **mirror** of each JSON Schema, pinned to the
   producer commit it was taken from, and a test that fails when the mirror and the producer
   disagree.
2. Contracts with no single producer (module catalog, Local-AI types, entitlements) are
   **defined in Zairenkai-Sdk** and consumed by everyone.
3. Zairenkai-Sdk's tests validate **captured outputs** (the fixtures listed above, with their
   recorded producer commit) against the schemas. Synthetic fixtures are allowed only when named
   synthetic.
4. Generated client types (Kotlin for App, Rust for Core) are optional and must be produced by a
   checked-in generator, never edited by hand.

## 3. Compatibility rules

- A version number changes only when a consumer would misread the new output. Adding an
  optional field does not change it; consumers ignore unknown fields (App uses
  `ignoreUnknownKeys`; Core uses serde defaults), except where a contract opts into
  `deny_unknown_fields` for safety (Tuning profiles).
- Removing a field, changing a type or a unit, or changing a field's meaning is a major change:
  the producer emits the new version only when asked, or keeps both for one release.
- Units travel with values (for example the `unit` field of every Tuning reading); a failed read
  is a status, never a substitute number.
- Request status and operation status are separate fields (Core, Tuning).
- Errors are structured: a stable `code` plus a human message; consumers branch on the code only.

## 4. Artifact versioning

| Artifact | Version source | Pinned by |
|---|---|---|
| zcored, zfasd, ztune components | crate version + source commit in `component.json` | Modules `components.lock.json` (full SHA) |
| `zairenkai` module zip | `components.lock.json` `module.version` (0.2.0 on the Modules topic branch) | App lock component `zairenkai-module` (full SHA of Modules) |
| zart, zthermal | crate version + commit | App lock (full SHA) |
| AFID asset | generated from the locked commit | App lock |
| APK | `versionName`/`versionCode` (0.1.0 / 1) | release tag (none yet) |

A pin moves only after the full-integration build passes at the new commit; a branch name is
never a pin.

## 5. Migrations

- FAS configuration has an in-code migration path to `SCHEMA_VERSION` (`config.rs`).
- The superseded modules `zairenkai_core` / `zairenkai_fas` are detected by the installer and
  listed as conflicts in the catalog; data moved to `/data/adb/zairenkai/{core,fas}`.
- No other contract has needed a migration yet.
