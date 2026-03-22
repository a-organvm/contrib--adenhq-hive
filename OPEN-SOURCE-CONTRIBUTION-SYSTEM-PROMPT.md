# ORGANVM Open-Source Contribution System — Operator Prompt

Drop this into a new session to initialize the cross-organ symbiotic contribution machine for any open-source project.

---

## Context

I run a 118-repo, 8-organ creative-institutional system called ORGANVM. The workspace is at `~/Workspace/`. Each organ is a GitHub organization with independent repos. The system has:

- **Promotion state machine**: LOCAL → CANDIDATE → PUBLIC_PROCESS → GRADUATED → ARCHIVED (forward-only, quality-gated)
- **Dependency graph**: I→II→III (unidirectional), IV orchestrates all, V observes, VII distributes
- **Governance**: `registry-v2.json` is single source of truth, `seed.yaml` per repo, `validate-deps.py` enforces edges
- **AI-conductor model**: Human directs, AI generates volume, human reviews

I contribute to open-source projects using a **Cross-Organ Symbiote** pattern where the contribution spans 5 organs simultaneously:

| Organ | Role | What lives here |
|-------|------|-----------------|
| **ORGAN-IV** (Taxis) | Orchestration hub | `contrib--{project}/` — journal, seed.yaml, plans, fork |
| **ORGAN-I** (Theoria) | Theory extraction | Formalized patterns extracted from the fusion of ORGANVM + target project |
| **ORGAN-II** (Poiesis) | Generative artifacts | Visualizations, diagrams, before/after comparisons |
| **ORGAN-III** (Ergon) | Shipped code | The actual PR payload — what lands upstream |
| **ORGAN-V** (Logos) | Public narrative | Essay/blog post telling the story of the contribution |

The lifecycle: `JOURNAL → EXTRACT → GENERATE → BUILD → SHIP → NARRATE` (IV→I→II→III→III→V). This is a cycle, not a pipeline — shipping feeds back into the journal.

## Prior Art

The first project using this system is AdenHQ/Hive (PR #6707 to `adenhq/hive`). The full implementation lives at `~/Workspace/organvm-iv-taxis/contrib--adenhq-hive/` with:
- Design spec: `.claude/plans/2026-03-21-hive-fusion-design.md`
- Implementation plan: `.claude/plans/2026-03-21-hive-fusion-implementation.md`
- Journal: `journal/2026-03-21-session-01.md`
- seed.yaml with produces/consumes edges

Use this as the structural template. Read it before starting a new contribution.

## What I Need You To Do

For the target open-source project I'll specify:

1. **Create the contribution workspace** at `~/Workspace/organvm-iv-taxis/contrib--{project}/`
2. **Fork the target repo** into my GitHub account (`4444J99/{repo}`)
3. **Deep-explore the codebase** — understand architecture, conventions, test patterns, CI, contribution rules
4. **Identify the highest-impact issue** — where ORGANVM patterns (governance, state machines, multi-agent coordination, versioning, testing infrastructure) map onto the target project's open problems
5. **Design the fusion** — what flows from ORGANVM into their codebase, what flows back. This is symbiotic, not extractive.
6. **Build Phase 1** — TDD, following their patterns exactly. The code must read as native to their codebase — ORGANVM concepts, their idiom.
7. **Wire the ORGANVM infrastructure** — GitHub repo, superproject submodule, registry-v2.json entry, seed.yaml, validate-deps clean
8. **Claim the issue, submit the PR**
9. **Write the journal entry** — decisions, artifacts, observations, next steps
10. **Give me the status audit** — every layer checked, nothing assumed

## Key Constraints

- **CONTRIBUTING.md is law** — read it before touching anything. Follow their issue claim process, commit conventions, test requirements, CI checks.
- **The code must be idiomatic to THEIR codebase** — not a foreign import. Study their patterns, mirror their file structure, use their tools.
- **Forward-only lifecycle** — versions, promotions, state transitions only go forward. No back-edges. This is Article VI.
- **Never overwrite production data** — read before write. The registry has a 50-repo safety check.
- **`uv` for Python unless told otherwise** — many modern projects use it
- **seed.yaml is required** — every contribution workspace gets a formal contract
- **Journal is the audit trail** — every session gets a dated entry

## ORGANVM Infrastructure Checklist

After building the PR, wire:

- [ ] GitHub repo: `organvm-iv-taxis/contrib--{project}`
- [ ] Push local repo to remote
- [ ] Add as submodule in superproject (`~/Workspace/organvm-iv-taxis/`)
- [ ] Register in `registry-v2.json` (via `~/Workspace/meta-organvm/organvm-corpvs-testamentvm/`)
- [ ] Validate deps: `python3 scripts/validate-deps.py` (0 new violations)
- [ ] Update CLAUDE.md organ map (repo count, new entry)
- [ ] Push superproject

## Key Repos for Reference

- **agentic-titan** (`~/Workspace/organvm-iv-taxis/agentic-titan/`) — Multi-agent orchestration framework. State machines, assembly dynamics, fission-fusion, checksummed snapshots. Source of governance patterns.
- **orchestration-start-here** (`~/Workspace/organvm-iv-taxis/orchestration-start-here/`) — Registry, governance-rules.json, validate-deps.py, promotion pipeline.
- **contrib--adenhq-hive** (`~/Workspace/organvm-iv-taxis/contrib--adenhq-hive/`) — First contribution using this system. Structural template.
- **registry-v2.json** (`~/Workspace/meta-organvm/organvm-corpvs-testamentvm/registry-v2.json`) — Single source of truth for all repos.

## Start Command

Tell me the target project (GitHub URL or name) and I'll run the full machine.
