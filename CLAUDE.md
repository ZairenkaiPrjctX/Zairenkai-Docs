# CLAUDE.md: Zairenkai-Docs

Instructions for Claude Code in `ZairenkaiPrjctX/Zairenkai-Docs`. Keep this file under 200 lines;
put detail in `AGENT/`.

@AGENTS.md
@AGENT/PROJECT_CONTEXT.md

## Current state (read first)

This repository holds the **ecosystem documentation** (Markdown only, no build): audit,
architecture, contracts, roadmap with the session checkpoint, and threat model (`README.md`).
It has no code, so no build or test command exists.

- Every status names its evidence (source, command output, CI run id) or says Unverified.
- Update `roadmap/IMPLEMENTATION_ROADMAP.md`'s session checkpoint at the end of every session.
- Component details stay in their repositories; link to them instead of copying.

## Commands

| Purpose | Command | Status |
|---|---|---|
| Build | — | Not Configured |
| Test | — | Not Configured |
| Lint / format | — | Not Configured |
| Validate agent foundation | `python3 <path-to-.github>/agent-foundation/scripts/validate_agent_setup.py .` | Configured |

Only list commands that exist in this repository. Never invent one.

## Working rules

- Inspect the relevant code before editing it. Preserve unrelated and uncommitted changes.
- Generated files are edited at their source, then regenerated. Managed foundation files
  (listed in `AGENT/.foundation-manifest.json`) are edited in `ZairenkaiPrjctX/.github`, not here.
- Destructive or outward-facing actions (deleting data, force-push, history rewrites,
  publishing releases, deploying, flashing devices) need explicit human approval each time.
- Do not claim a build, test, or signature succeeded without showing the output.

## Git

- Work on a topic branch; open **draft** pull requests; never merge or approve.
- Commits must be cryptographically signed. If signing fails, stop and report the error.
  Trailer format and verification: `AGENT/policies/git-and-commit-signing.md`.

## Where things are

- `AGENT/README.md`: index of the agent library (prompts, policies, playbooks, templates).
- `.claude/skills/`: run `/repo-audit`, `/implement-feature`, `/bug-fix`, `/test-and-verify`,
  `/security-review`, `/release-readiness`.
- `.claude/rules/`: path-scoped rules, loaded automatically.
