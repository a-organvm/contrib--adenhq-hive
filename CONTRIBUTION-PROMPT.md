# Hive Contribution Prompt

Use this prompt when starting a session in this directory to work on AdenHQ/Hive PRs.

---

## Context

I'm contributing to adenhq/hive (https://github.com/adenhq/hive) — a YC-backed open-source framework for autonomous, adaptive AI agents. 9,600+ stars, Python, Apache 2.0. The CEO (Vincent Jiang) reached out to me directly via email on Feb 26, inviting me to their Discord and asking for open-source contributions. I replied March 19. Their team (5 members) starred my portfolio on GitHub.

I want to make a meaningful first contribution. My background:
- 113-repo multi-agent orchestration system (ORGANVM)
- Promotion state machine with forward-only transitions, 50 dependency edges, 0 violations
- 23,470 tests across the system, 3,266 in the application pipeline alone
- 104 CI/CD pipelines
- MCP server infrastructure (filesystem, memory, sequential thinking)
- Deep Python, testing, CI/CD, documentation, security experience
- 739K words of documentation governance
- 100+ courses taught

## Target Issues (pick one or propose your own)

1. **#2805** — Expand integrations/tools ecosystem. CONTRIBUTING.md explicitly calls this out as what they need. I could add an MCP-based integration or a new tool server.

2. **#6613** — Improve reproducibility and versioning for self-evolving agents. My promotion state machine (LOCAL → CANDIDATE → PUBLIC_PROCESS → GRADUATED → ARCHIVED) with forward-only transitions and audit trails maps directly. Could contribute a versioning/snapshot system.

3. **#6612** — Enable parallel test execution with pytest-xdist. I run 3,266 tests in my own pipeline and have deep pytest infrastructure experience.

4. **#6646** — Security: No authentication on HTTP API. Security implementation is in my wheelhouse.

5. **Documentation** — No assignment needed per CONTRIBUTING.md. Could improve MCP integration docs (they have MCP_INTEGRATION_GUIDE.md, MCP_SERVER_GUIDE.md, MCP_BUILDER_TOOLS_GUIDE.md), add examples, or strengthen the roadmap docs.

## Approach

1. Fork adenhq/hive to 4444J99/hive
2. Clone into this directory: `git clone https://github.com/4444J99/hive.git repo`
3. Read CONTRIBUTING.md, CLAUDE.md, AGENTS.md for conventions
4. Run `./quickstart.sh` for setup
5. Run `make check` (ruff) and `make test` (pytest) to verify local setup
6. Pick the issue, claim it with a comment on GitHub, wait for assignment (or go docs route for no-assignment-needed)
7. Create feature branch: `git checkout -b feature/your-feature-name`
8. Implement, test, submit PR

## Rules from CONTRIBUTING.md

- Must claim issue before PR (comment "I'd like to work on this!" and wait for assignment within 24h)
- **Exceptions:** docs fixes and micro-fixes (<20 lines, no logic changes) don't need assignment
- Run `make check` (ruff check + ruff format --check on core/ and tools/) before submitting
- Run `make test` (cd core && pytest tests/ -v) before submitting
- Follow commit conventions
- PRs for unassigned issues may be delayed/closed

## Repo Structure

```
adenhq/hive/
├── core/           # Framework core (Python)
│   ├── framework/  # Agent framework
│   ├── frontend/   # UI
│   ├── examples/   # Example agents
│   └── tests/      # Core tests
├── tools/          # Tool servers and integrations
│   ├── src/        # Tool implementations
│   └── tests/      # Tool tests
├── docs/           # Documentation
│   ├── roadmap.md  # Product roadmap
│   └── i18n/       # Translations
├── CONTRIBUTING.md # Contribution guidelines
├── CLAUDE.md       # Claude Code instructions
├── AGENTS.md       # Agent instructions
└── Makefile        # make check, make test
```

## Cross-References

- **Pipeline origin:** `~/Workspace/4444J99/application-pipeline/applications/2026-03-18/outreach-plan-doubling-back.md` (AdenHQ, entry #1)
- **Contact CRM:** Vincent Jiang (CEO), Timothy Zhang (Co-founder), Adel Burieva (Co-founder) in `signals/contacts.yaml`
- **Network graph:** AdenHQ node with 5 edges in `signals/network.yaml`
- **Email thread:** Gmail thread `19c9c567bc1b9a26` — Vincent's outreach + my reply
