# Threat model

Ecosystem-level threats across the app, root daemons, the root module, signed catalogs, CI,
the optional server and firmware metadata. Component threat models stay authoritative for their
details (Core `docs/security/THREAT_MODEL.md`, ThermalTools `docs/SAFETY.md`, AFID
`docs/LICENSE_AND_DISTRIBUTION.md`). Status words: **mitigated in code** (a control exists and is
tested on a host), **open** (no control yet), **unverified on device** (every on-device claim).

## 1. Assets

| Asset | Why it matters |
|---|---|
| Root on the user's device | zcored, zfasd and ztune run as root; a bad write can overheat, brick or drain a device |
| Hardware safety margins | thermal protection, frequency limits, GPU tables |
| Module-signing private key, APK keystore | they authorize code that runs as root on users' devices |
| Locked source commits | they define what ships |
| User privacy | device identifiers, installed packages, diagnostics |
| Licence compliance | GPL-3.0 Tuning beside Apache-2.0 components; CC BY-SA device data; vendor firmware rights |

## 2. Trust boundaries

1. App (`untrusted_app`) ↔ root, crossed only by `su -c` with a typed argv after the user enables
   root.
2. Root daemons ↔ kernel nodes (`/sys`, `/proc`).
3. Core ↔ components (FAS socket, `execve` of zthermal and ztune with fixed argv).
4. APK ↔ module zip (catalog signature, per-file digests).
5. CI ↔ secrets (only `release.yml`, `release` environment, deployable from `dev`).
6. AFID ↔ external firmware sources (disabled until reviewed).
7. Future: App ↔ local model (untrusted output), App ↔ license server (network).

## 3. Threats and controls

| # | Threat | Control | Status |
|---|---|---|---|
| T1 | The app is tricked into running an arbitrary root command | Root commands are `ShellScript`s of validated `ShellWord`s; no string shell; no `su` before consent | mitigated in code (App tests) |
| T2 | A compromised or buggy client asks Core for an unsafe change | Deny-by-default policy, typed operations only, compiled-in safety rules, raising actions refused at thermal SEVERE, read-back of every change | mitigated in code (Core pipeline tests) |
| T3 | A daemon writes outside its bounds | FAS: floors within `[cpuinfo_min, scaling_max]`, restore-before-write, lease; ztune: advertised values ∩ Tier D grant, journal, rollback; ThermalTools: `denied_target`, empty validated allowlist | mitigated in code; unverified on device |
| T4 | A family/SoC match authorizes a write on the wrong device | Tuning resolver grants writes only to an exact device/build/kernel `device_validated` profile with hardware evidence; none ships | mitigated in code (tests) |
| T5 | A tampered module zip is installed | Catalog signed with a key in the reviewed trust list; per-file digests; the root manager's own installer; confirmation | mitigated in code; signing never exercised (F-03) |
| T6 | The module-signing key or keystore leaks | Keys only in the `release` environment; written to `$RUNNER_TEMP` mode 600 and removed; verifier rejects key material in APKs | mitigated in workflow; required reviewers unavailable on the plan (branch policy instead) |
| T7 | CI is abused to exfiltrate secrets | `ci.yml` holds no secret since `e0d53bb`; `permissions: contents: read`; actions pinned by SHA; `persist-credentials: false` | mitigated |
| T8 | A locked dependency is swapped | Full 40-character SHAs; `verify_integration.py build` checks each artifact's SHA-256 and the checkout HEAD | mitigated |
| T9 | Stale or false status shown to users | No mocks in production; unprobed capability = NOT_VERIFIED; Core reports Tuning read-only without authorization | mitigated in code |
| T10 | GPL obligations broken | ztune is a separate process; Corresponding Source and licence in the zip (`assemble.py` refuses otherwise); no GPL code in Apache repositories | mitigated in code |
| T11 | Firmware payloads or restricted data published | AFID JSON-only catalog, default `unknown` distribution, SHA-256 allowlist, sources disabled until review, no S3 publish without a flag | mitigated in code |
| T12 | Local model output turns into a privileged action | Planned: closed typed `ToolRegistry`, Core revalidates every request, user approval per policy, audit event; no remote AI API | **open** (Intelligence-Engine empty) |
| T13 | Privacy leak in diagnostics | zart reports no device identifiers; fixtures recorded host identity once and were not committed (App verification report) | partly mitigated; exports not reviewed |
| T14 | License server outage blocks the device | Design rule: local diagnostics never depend on it | **open** (no server) |
| T15 | Root manager differences (Magisk, KernelSU, APatch) cause a broken install | Commands read from each manager's source; single module; uninstall restores FAS and ztune state | unverified on device |

## 4. Residual risks requiring a person

- Physical-device validation of every write path (FAS floors, ztune writes once a Tier D
  profile exists, power mode) and of install/uninstall on each root manager.
- The first signed release and verification that the stored key matches the trusted public key.
- Review of AFID source terms (GitLab robots.txt, tadiphone) before enabling any source.
