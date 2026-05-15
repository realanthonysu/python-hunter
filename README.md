# python-hunter

> Python-specific software engineering skills for AI-assisted coding — because "running ≠ done".

English | [中文](./README_cn.md)

![License](https://img.shields.io/badge/license-Apache--2.0-green)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)
![Python](https://img.shields.io/badge/Python-3.10+-3776ab)

An [Agent Skills](https://agentskills.io/specification)-compatible skill suite that enforces Python-specific software engineering discipline in AI-assisted coding. Built on Karpathy's insight: *thinking can be outsourced, understanding cannot*.

These skills were originally part of [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian) and have been extracted into a dedicated Python-focused repository.

## Skills in this Repository

| Skill | Description | Scope |
|-------|-------------|-------|
| **vibe-coding-coach** | Python engineering discipline — maps 5 iron rules to ruff/mypy/pytest/bandit+semgrep tool chain | Python development |
| **vibe-coding-scaffold** | Project scaffolding — standardized Python dev environment with uv/ruff/mypy/ty/pytest/prek | Python project setup |

`vibe-coding-coach` defines *how* to verify in Python. `vibe-coding-scaffold` ensures the verification infrastructure is in place from day one.

> The language-agnostic core skill (`vibe-coding-guardian`) remains in its own [repository](https://github.com/realanthonysu/vibe-coding-guardian).

---

## Highlights

### vibe-coding-coach (v1.1)

- **Dual mode** — 🚀 Prototype for speed (Jupyter-friendly), 🏭 Production for safety (PyPI-ready)
- **Python-specific risk keywords** — auto-detects `eval()`, `pickle.loads()`, `subprocess(shell=True)`, and more
- **Tool chain auto-detection** — respects your project's existing tool choices (ruff vs flake8, pytest vs unittest)
- **5 computable signals** — replaces "gut feeling" with `git diff` + `grep`

### vibe-coding-scaffold (v0.2)

- **Zero decision fatigue** — one opinionated toolchain, no configuration debates
- **3 project types** — library (PyPI), web-api (FastAPI), data-pipeline (ETL)
- **Incremental supplement** — adds missing configs to existing projects without overwriting
- **Python version parameterization** — supports any target Python version

---

## Table of Contents

- [python-hunter](#python-hunter)
  - [Skills in this Repository](#skills-in-this-repository)
  - [Highlights](#highlights)
    - [vibe-coding-coach (v1.1)](#vibe-coding-coach-v11)
    - [vibe-coding-scaffold (v0.2)](#vibe-coding-scaffold-v02)
  - [Table of Contents](#table-of-contents)
  - [Python Tool Chain](#python-tool-chain)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [Skill CLI (Recommended)](#skill-cli-recommended)
    - [Claude Code](#claude-code)
    - [Other Agents](#other-agents)
  - [Usage](#usage)
    - [vibe-coding-coach](#vibe-coding-coach)
    - [vibe-coding-scaffold](#vibe-coding-scaffold)
  - [File Structure](#file-structure)
  - [Evolution History](#evolution-history)
    - [vibe-coding-coach (v1.1)](#vibe-coding-coach-v11-1)
    - [vibe-coding-scaffold (v0.2)](#vibe-coding-scaffold-v02-1)
  - [License](#license)

---

## Python Tool Chain

Both skills share a common, opinionated toolchain:

| Role | Tool | Why |
|------|------|-----|
| Package manager | **uv** | Fast, Rust-based, becoming the Python standard |
| Linter + Formatter | **ruff** | Replaced flake8, isort, black — one tool for all |
| Type checker (deep) | **mypy** | Industry standard, catches subtle type bugs |
| Type checker (fast) | **ty** | Astral's new Rust-based checker, instant feedback |
| Testing | **pytest** | De facto standard |
| Coverage | **coverage** | Measures test completeness |
| Security scan | **bandit** + **semgrep** | Python-specific SAST + OWASP Top 10 |
| Git hooks | **prek** | Rust-based pre-commit replacement, fast and dependency-free |

---

## Prerequisites

- **Claude Code** (latest version), **Trae**, or any **Agent Skills-compatible** agent
- **Python 3.10+** (for coach) or **Python 3.12+** (for scaffold default)
- **Git** (optional — skills work without git history, with graceful degradation)

---

## Installation

### AI Agent Install (Recommended)

Paste this to your AI agent (Claude Code, Trae, Cursor, etc.):

```
Install the python-hunter skills for me:

1. Clone https://github.com/realanthonysu/python-hunter into a temp directory.
2. Copy skills/vibe-coding-coach and skills/vibe-coding-scaffold into
   your user-level skills directory (e.g. ~/.claude/skills/ for Claude Code).
3. Verify that SKILL.md and references/ are present in each skill folder.
4. Confirm the install paths when done.
```

### Skill CLI

```bash
# Install coach (Python engineering discipline)
npx skills add realanthonysu/vibe-coding-coach

# Install scaffold (Python project scaffolding)
npx skills add realanthonysu/vibe-coding-scaffold
```

### Claude Code (Manual)

**Project-level** (follows the project):

```bash
mkdir -p .claude/skills
cp -r skills/vibe-coding-coach .claude/skills/
cp -r skills/vibe-coding-scaffold .claude/skills/
```

**User-level** (available in all projects):

```bash
mkdir -p ~/.claude/skills
cp -r skills/vibe-coding-coach ~/.claude/skills/
cp -r skills/vibe-coding-scaffold ~/.claude/skills/
```

### Other Agents

This skill follows the [Agent Skills specification](https://agentskills.io/specification). Place the skill folders in your agent's skill directory (e.g., `.agents/skills/`).

---

## Usage

### vibe-coding-coach

No explicit activation needed — the skill auto-activates on any Python code generation, modification, or refactoring task.

```
You: Add a CSV export endpoint to our FastAPI app

AI: [auto-activates vibe-coding-coach]

🟡 Medium risk (new feature + file I/O). Need to confirm:
1. Expected data volume — streaming needed for large datasets?
2. Encoding — UTF-8 or need BOM for Excel compatibility?
3. Filtering — export all records or with query params?

You: Up to 100k rows, UTF-8, with date range filter

AI: [generates streaming CSV + Pydantic query params + error handling]
[ruff check ✅] [mypy ✅] [pytest ✅]
Completion: ✅ positive path ✅ negative path ✅ no regression ✅ contract consistent
```

### vibe-coding-scaffold

Trigger with a slash command or natural language:

```
/vibe-coding-scaffold

or

"帮我初始化一个 FastAPI 后端项目"
"Create a new Python library for data validation"
"给这个项目加上工程规范配置"
```

---

## File Structure

```
python-hunter/
├── README.md                           # This file
├── README_cn.md                        # Chinese documentation
├── LICENSE                             # Apache-2.0
├── CHANGELOG.md                        # Version history
└── skills/
    ├── vibe-coding-coach/         # Python engineering discipline
    │   ├── SKILL.md                    # Core skill (iron rules + decision flow)
    │   ├── README.md                   # English documentation
    │   └── references/                 # On-demand deep guides
    │       ├── architecture.md
    │       ├── security.md
    │       ├── quality-gates.md
    │       ├── refactoring.md
    │       └── examples.md
    └── vibe-coding-scaffold/             # Python project scaffolding
        ├── SKILL.md                    # Core skill (decision flow + templates)
        ├── README.md                   # English documentation (with Chinese summary)
        └── references/                 # Project type templates
            ├── library.md
            ├── web-api.md
            └── data-
---

## Evolution History

### vibe-coding-coach (v1.1)

| Version | Key Change |
|---------|-----------|
| v1.0.0 | Initial release — Python-specific edition of vibe-coding-guardian, maps iron rules to ruff/mypy/ty/pytest/bandit/uv tool chain |
| v1.1.0 | ty downgrade to optional (Beta, ~53% spec conformance), Semgrep added to security stack, ML deserialization risk keywords (torch.load/joblib.load), Astral acquisition risk note |

### vibe-coding-scaffold (v0.2)

| Version | Key Change |
|---------|-----------|
| v0.1.0 | Initial release — opinionated Python project scaffolding with uv/ruff/mypy/ty/pytest/coverage/prek, supports library/web-api/data-pipeline project types |
| v0.2.0 | Python version parameterization, prek hook version placeholders (query latest at generation time), py.typed marker for library projects, Astral acquisition risk note |

For detailed changes, see [CHANGELOG.md](./CHANGELOG.md).

---

## License

[Apache-2.0](./LICENSE)
