# QWEN.md — python-hunter

## Project Overview

**python-hunter** is a collection of [Agent Skills](https://agentskills.io/specification) for Python-specific software engineering discipline. This is a **documentation-only repository** — it contains no Python source code, build system, or runtime. All content is Markdown defining AI agent behaviors.

The repository provides two skills that enforce Python engineering discipline in AI-assisted coding, built on Karpathy's principle: *"thinking can be outsourced, understanding cannot."*

### Skills

| Skill | Purpose | Version |
|-------|---------|---------|
| **vibe-coding-coach** | Python engineering discipline — 5 iron rules mapped to ruff/mypy/pytest/bandit/semgrep toolchain. Provides risk-graded pre-generation understanding, during-generation verification, and post-generation completion gates. | v1.1 |
| **vibe-coding-scaffold** | Python project scaffolding — standardized dev environment with uv/ruff/mypy/ty/pytest/coverage/prek. Supports library, web-api, data-pipeline, and CLI project types. | v0.2 |

Both skills follow the Agent Skills specification with 3-tier progressive disclosure (SKILL.md → references/) to minimize token usage.

## Repository Structure

```
python-hunter/
├── README.md / README_cn.md       # Bilingual documentation
├── CHANGELOG.md                   # Version history
├── LICENSE                        # Apache-2.0
├── CLAUDE.md                      # Claude Code context
├── skills/
│   ├── vibe-coding-coach/
│   │   ├── SKILL.md               # Core skill: iron rules, decision flow, gate functions
│   │   ├── README.md
│   │   └── references/            # On-demand deep guides
│   │       ├── architecture.md    # Module boundaries, dependency management, layered architecture
│   │       ├── security.md        # Input validation, auth, data protection, ML deserialization
│   │       ├── quality-gates.md   # Five-stage verification, pytest strategy, prek hooks
│   │       ├── refactoring.md     # Refactoring patterns, code smells, safety nets
│   │       └── examples.md        # Python正反 examples for all 5 iron rules
│   └── vibe-coding-scaffold/
│       ├── SKILL.md               # Core skill: decision flow, templates, gap analysis
│       ├── README.md
│       └── references/            # Project type templates
│           ├── library.md         # PyPI package template
│           ├── web-api.md         # FastAPI service template
│           ├── data-pipeline.md   # ETL pipeline template
│           └── cli.md             # CLI tool template
```

## Key Design Decisions

- **SKILL.md is the source of truth** — YAML frontmatter contains metadata; body contains decision flows and rules
- **references/ uses progressive disclosure** — files are only loaded when the task requires deep domain knowledge
- **coach has dual-mode architecture** — Prototype (speed-first) and Production (safety-first)
- **Toolchain is opinionated but respectful** — coach detects existing project tools before applying its own; scaffold uses a fixed toolchain for new projects
- **Skills were extracted from** [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian) — the language-agnostic core remains there

## Development Workflow

Since this is a documentation repository, typical tasks involve:

1. **Editing SKILL.md** — updating decision flows, iron rules, gate functions, or metadata
2. **Adding/updating reference guides** — in `references/` directories
3. **Updating README files** — keeping English and Chinese versions in sync
4. **Modifying CHANGELOG.md** — recording version updates following existing format
5. **Validating YAML frontmatter** — ensuring metadata (name, description, version, triggers) is correct

### Editing Conventions

- SKILL.md files use YAML frontmatter followed by Markdown body
- Reference files are self-contained guides for specific domains
- Decision flows use ASCII art diagrams (box-and-arrow style)
- Tables are used extensively for risk matrices and tool mappings
- Code examples use Python-specific syntax with tool commands

## Coach: Five Iron Rules Summary

| Rule | Principle | Gate Function |
|------|-----------|---------------|
| 铁律一 | Understand before generating | Risk-graded understanding confirmation (🟢🟡🔴) |
| 铁律二 | Running ≠ Complete | Completion self-check (positive/negative/regression/contract) |
| 铁律三 | Verification structure > quantity | Key business rule coverage test |
| 铁律四 | Higher risk → deeper verification | Risk level assessment with Python-specific keywords |
| 铁律五 | Know when to escalate | Escalation signal detection (coverage, impact, TODOs, contract, security) |

## Scaffold: Supported Project Types

| Type | Description | Key Features |
|------|-------------|--------------|
| **library** | Publishable PyPI package | hatchling build, py.typed marker, full metadata |
| **web-api** | FastAPI web service | Pydantic config, health check, .env.example |
| **data-pipeline** | ETL/data processing | data/ directories, minimal deps |
| **cli** | Command-line tool | click/typer entry point, script distribution |

## Important Context

- **v1.1 changes**: ty downgraded to optional (Beta, ~53% typing spec conformance), Semgrep added to security stack, ML deserialization risk keywords added
- **v0.2 changes**: Python version parameterization, prek hook version placeholders (query latest at generation time), py.typed marker for library projects
- **Ecosystem risk**: Astral (uv, ruff, ty parent company) was acquired by OpenAI in March 2026 — noted in both skills' Gotchas sections
- **Rename history**: `vibe-coding-pyguardian` → `vibe-coding-coach`, `vibe-coding-pyinit` → `vibe-coding-scaffold`

## Python Toolchain (Shared by Both Skills)

| Role | Tool | Notes |
|------|------|-------|
| Package manager | **uv** | Rust-based, becoming Python standard |
| Linter + Formatter | **ruff** | Replaces flake8 + isort + black |
| Type checker (deep) | **mypy** | Industry standard, strict mode recommended |
| Type checker (fast) | **ty** | Optional/Beta, Rust-based, 10-60x faster |
| Testing | **pytest** | De facto standard |
| Coverage | **coverage** | Measures test completeness |
| Security scan | **bandit** + **semgrep** | Python SAST + OWASP Top 10 |
| Git hooks | **prek** | Rust-based pre-commit replacement |

## Related Repositories

| Repository | Relationship |
|------------|-------------|
| [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian) | Language-agnostic core — defines *what* to verify |
| [vibe-testing](https://github.com/realanthonysu/vibe-testing) | Testing skills (testsmith, testweaver) |
