# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

python-hunter is a collection of [Agent Skills](https://agentskills.io/specification) for Python-specific software engineering discipline. This is a **documentation-only repository** — no Python code, no build system, no tests, no CI/CD. All content is Markdown.

Two skills are maintained here:
- **vibe-coding-coach** — Python engineering discipline with 5 iron rules, dual modes (Prototype/Production), and toolchain integration (ruff/mypy/pytest/bandit/semgrep)
- **vibe-coding-scaffold** — Python project scaffolding with opinionated toolchain (uv/ruff/mypy/ty/pytest/coverage/prek)

Both follow the [Agent Skills specification](https://agentskills.io/specification) with 3-tier progressive disclosure to minimize token usage.

## Repository Structure

```
skills/
├── vibe-coding-coach/
│   ├── SKILL.md           # Core skill definition (~450 lines) — iron rules, decision flow, gate functions
│   ├── README.md          # English documentation
│   └── references/        # On-demand deep guides (loaded only when task touches relevant domain)
│       ├── architecture.md
│       ├── examples.md
│       ├── quality-gates.md
│       ├── refactoring.md
│       └── security.md
└── vibe-coding-scaffold/
    ├── SKILL.md           # Core skill definition (~350 lines) — scaffolding flow, templates
    ├── README.md          # English documentation
    └── references/        # Project type templates
        ├── data-pipeline.md
        ├── library.md
        └── web-api.md
```

Root files: `README.md`, `README_cn.md` (Chinese), `CHANGELOG.md`, `LICENSE` (Apache-2.0)

## Key Design Decisions

- **SKILL.md is the source of truth** — YAML frontmatter contains metadata (name, description, version, triggers); body contains the decision flow and rules
- **references/ files are progressive disclosure** — only load when the task requires deep domain knowledge (architecture, security, quality gates, etc.)
- **Skills are language-agnostic in structure** but Python-specific in content — coach maps 5 iron rules to Python tooling; scaffold scaffolds Python projects
- **Dual mode architecture** in coach: Prototype (speed-first, Jupyter-friendly) and Production (safety-first, PyPI-ready)
- **Toolchain is opinionated but respectful** — coach detects existing project tools before applying its own; scaffold uses a fixed toolchain

## Common Development Tasks

Since this is a documentation repository, typical tasks involve:
- Editing SKILL.md files to update decision flows, iron rules, or gate functions
- Adding/updating reference guides in `references/` directories
- Updating README.md files (English/Chinese)
- Modifying CHANGELOG.md for version updates
- Ensuring YAML frontmatter in SKILL.md files is valid and triggers are correctly specified

## Important Context

- The skills were extracted from [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian) — the language-agnostic core remains there
- coach v1.1 added: ty downgrade to optional (Beta), semgrep to security stack, ML deserialization risk keywords
- scaffold v0.2 added: Python version parameterization, prek hook version placeholders, py.typed marker
- Astral (uv, ruff, ty parent company) was acquired by OpenAI in March 2026 — noted as ecosystem risk in skills
- The repository has zero commits and all files are currently untracked (freshly initialized)
