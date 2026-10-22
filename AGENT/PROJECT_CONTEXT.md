# Project context: Zairenkai-Docs

> Owned by this repository. Update it whenever verified facts change. Label every statement
> **Verified** (with `Evidence:`), **Unverified**, or **Unknown / Not Configured**.

## Identity

| Field | Value | Status |
|---|---|---|
| Repository | `ZairenkaiPrjctX/Zairenkai-Docs` | Verified. Evidence: GitHub API |
| GitHub description | (none set) | Verified. Evidence: GitHub API |
| Visibility | public | Verified. Evidence: `list_repos`, 2026-10-22 |
| Default branch | `dev` | Verified. Evidence: `git ls-remote`, 2026-10-22 |
| Purpose | Ecosystem documentation: audit, architecture, contracts, roadmap, threat model | Verified. Evidence: `README.md`, 2026-10-22 |

## Contents

| Area | Location | Status |
|---|---|---|
| Ecosystem audit, architecture, responsibility matrix, contracts, roadmap, threat model | `audit/`, `architecture/`, `roadmap/`, `security/` | Verified. Evidence: files present, written from the audit of 2026-10-22 |
| Build, tests | — | Not Configured (Markdown only) |

## Authoritative sources

| Topic | Source |
|---|---|
| Repository facts | this file |
| Architecture | `AGENT/ARCHITECTURE.md` |
| Organization standards | `ZairenkaiPrjctX/.github` → `agent-foundation/standards/` |
| Repository list and relationships | `ZairenkaiPrjctX/.github` → `agent-foundation/inventory/repositories.json` |

## Related repositories

The documents describe the 16 Zairenkai repositories; their facts come from the audit of
2026-10-22 (`audit/ECOSYSTEM_AUDIT.md`).

## Open questions to answer when code lands

- Which documentation tool (if any) builds the docs, and where is it published?
- Which repositories' behaviour do these docs describe, and who keeps them in sync?
