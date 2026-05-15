# Skill Design Review: Spec Compliance & Claude Code Compatibility

> Date: 2026-05-14
> Evaluated against: [Agent Skills Specification](https://agentskills.io/specification) + [Claude Code Skills Docs](https://code.claude.com/docs/en/skills)

## 1. Spec Compliance Checklist

### 1.1 Frontmatter Fields

| Requirement | vibe-coding-coach | vibe-coding-scaffold | Verdict |
|-------------|:-:|:-:|---------|
| `name` required, max 64 chars, lowercase+hyphens | `vibe-coding-coach` | `vibe-coding-scaffold` | Both pass |
| `name` matches directory name | Yes | Yes | Both pass |
| `description` required, max 1024 chars | **~750 chars** | **~400 chars** | Both pass spec limit |
| `license` optional | `Apache-2.0` | Missing | Scaffold should add |
| `compatibility` optional | Missing | Missing | OK, optional |
| `metadata` optional | Has `author`, `version`, `paradigm`, `core-thesis`, `language`, `toolchain` | Missing entirely | Scaffold should add `metadata` |
| `allowed-tools` optional (experimental) | Missing | Missing | OK, optional |
| No XML angle brackets in frontmatter | Clean | Clean | Both pass |
| Frontmatter starts on line 1 with `---` | Yes | Yes | Both pass |

### 1.2 Directory Structure

| Requirement | coach | scaffold | Verdict |
|-------------|:-----:|:--------:|---------|
| `SKILL.md` present (case-sensitive) | Yes | Yes | Both pass |
| `references/` directory | 5 files | 3 files | Both pass |
| No unexpected top-level files | Clean | Clean | Both pass |
| `README.md` (not required but present) | Yes | Yes | Good practice |

### 1.3 Body Content

| Requirement | coach | scaffold | Verdict |
|-------------|:-----:|:--------:|---------|
| Markdown format | Yes | Yes | Both pass |
| Clear instructions for the agent | Yes — decision flow, gate functions, iron rules | Yes — step-by-step scaffolding flow | Both pass |
| Under 500 lines (recommended) | 449 lines | 349 lines | Both pass |

## 2. Claude Code Specific Compatibility

### 2.1 Description as Trigger (Critical)

Claude Code scans **only the frontmatter `description`** (~100 tokens) to decide whether to load a skill. The description IS the trigger mechanism.

**vibe-coding-coach:**
```
description: >
  Use when: (1) starting any non-trivial Python coding task to assess risk level
  before implementation; (2) about to claim Python work is complete — verify with
  ruff + mypy + pytest evidence; (3) touching security-sensitive Python code (auth,
  payments, encryption, pickle, subprocess, eval); (4) diff impact radius unclear
  and blast analysis needed; (5) deciding whether a Python change requires human
  review. Also triggers on: risk assessment, blast radius, escalation, "should I
  proceed", 风险评估, 升级判断, 完成条件, 安全审查, entropy drift, green-light
  illusion, Python-specific: type hints, virtualenv, pyproject.toml, async safety.
```

- **Strength**: Very explicit about when to activate. The 5-scenario structure gives Claude clear matching patterns.
- **Strength**: Bilingual triggers (English + Chinese) expand matching surface.
- **Risk**: ~750 chars. Claude Code truncates `description + when_to_use` at **1,536 characters** in the skill listing. Currently safe, but adding `when_to_use` could push it over.
- **Issue**: Does not use the Claude Code-specific `when_to_use` field. Per spec, this field is "appended to description in the skill listing" and provides a cleaner separation of concerns.

**vibe-coding-scaffold:**
```
description: >
  Use when initializing a new Python project or adding best-practice tooling to an existing one.
  Scaffolds a standardized local development environment with uv, ruff, mypy, ty, pytest, coverage, and prek.
  Triggers on: "初始化 Python 项目", "create a Python project", "scaffold Python", "新建 Python 项目",
  "set up Python dev environment", "给项目加上工程规范", "初始化 FastAPI 项目", "new FastAPI project",
  "Python 项目脚手架", "pyscaffold", "scaffold", "pyinit", or any request to set up a Python project with best practices.
```

- **Strength**: Clear "Use when" + explicit trigger phrases. Well-structured.
- **Strength**: ~400 chars, leaves headroom.
- **Issue**: Same — no `when_to_use` field.

### 2.2 Missing Claude Code Extensions

| Field | coach | scaffold | Impact |
|-------|:-----:|:--------:|--------|
| `when_to_use` | Missing | Missing | Medium — could improve trigger precision |
| `paths` | Missing | Missing | Medium — neither skill scopes to `*.py` files |
| `hooks` | Missing | Missing | Low — not needed for instruction-only skills |
| `shell` | Missing | Missing | Low — defaults to bash, works on Windows via WSL |

**`paths` consideration**: Both skills are Python-specific. Adding `paths: "**/*.py"` would restrict auto-activation to Python file contexts. However, this might be too narrow — coach should also activate on `pyproject.toml` changes, CI config, etc. Current "no paths" (activate on any file) may be the right call for coach.

**`when_to_use` opportunity**: Extracting trigger phrases from `description` into `when_to_use` would keep the description focused on "what" and let `when_to_use` handle "when":

```yaml
# Example improvement for coach
description: >
  Python engineering discipline — 5 iron rules mapped to ruff/mypy/pytest/bandit.
  Risk-graded pre-generation understanding, during-generation verification depth,
  post-generation completion gates and human escalation signals.
when_to_use: >
  Starting any Python coding task, about to claim Python work is complete,
  touching security-sensitive code (auth, payments, eval, pickle, subprocess),
  diff impact unclear, deciding if human review needed. Triggers: risk assessment,
  blast radius, escalation, 风险评估, 完成条件, 安全审查.
```

### 2.3 Progressive Disclosure Architecture

The 3-tier loading pattern is central to both the spec and Claude Code's performance model:

| Tier | When Loaded | coach | scaffold |
|------|------------|-------|----------|
| Tier 1: Frontmatter (~100 tokens) | Always, for skill selection | name + description | name + description |
| Tier 2: SKILL.md body | On activation | Decision flow, 5 iron rules, gate functions | Scaffolding steps, project types, common configs |
| Tier 3: references/*.md | Only when task requires | architecture, security, quality-gates, refactoring, examples | library, web-api, data-pipeline templates |

**Assessment**: Both skills follow progressive disclosure well.

- **Coach** has 5 reference files, each loaded only when the corresponding domain is touched. The SKILL.md body itself is ~450 lines — dense but focused. The references table at the bottom clearly documents when to load each file.
- **Scaffold** has 3 reference files (project type templates), loaded only after project type is confirmed. The SKILL.md body contains the decision flow and common configs, keeping templates in references.

**One concern**: Coach's body is 449 lines. While under the 500-line soft limit, it's dense with tables, ASCII art decision trees, and gate functions. Claude Code loads the entire body on activation. This is a tradeoff between "everything in one place" and context efficiency. The current design is defensible — the alternative (splitting iron rules into separate reference files) would add latency per iron rule evaluation.

## 3. Design Principles vs. Problem Solving

### 3.1 Does Coach Solve Its Problem?

**Problem**: Vibe coding's "completion conditions drift" — AI generates fast but verification is shallow or skipped.

| Design Element | How It Solves the Problem | Assessment |
|----------------|--------------------------|------------|
| 5 Iron Rules | Maps abstract discipline to computable gates | **Strong** — each rule has a concrete gate function with pass/fail criteria |
| Risk-graded approach (🟢🟡🔴) | Avoids over-verifying trivial changes, under-verifying critical ones | **Strong** — right-sized friction |
| Dual mode (Prototype/Production) | Respects vibe coding speed while enabling safety when needed | **Strong** — default Prototype = low friction, auto-upgrade prompts catch deployment risk |
| Gate Functions with specific tools | Makes verification reproducible, not "gut feeling" | **Strong** — `ruff check . → 0 errors` is computable |
| Escalation signals (Iron Rule 5) | Catches cases where automation isn't enough | **Strong** — 5 measurable signals, not vibes |
| Tool chain auto-detection | Respects existing projects, doesn't force tool changes | **Strong** — practical for real-world use |
| Python-specific risk keywords | Targets the most dangerous Python patterns | **Strong** — eval, pickle, subprocess, etc. |

**Weaknesses**:
- **Iron Rule 1's understanding template** adds conversational overhead. For low-risk tasks (🟢), the "one sentence intent" is fine. But for 🟡 tasks, asking "WHAT + CONSTRAINTS" before every change could feel like friction to a vibe coder who just wants to ship. The skill needs to be honest about this tradeoff.
- **"人工确认核心行为"** in Prototype mode (when no test suite exists) is acknowledged as a weak constraint in the Gotchas. This is an honest admission but means the skill's effectiveness degrades significantly on projects without tests.

### 3.2 Does Scaffold Solve Its Problem?

**Problem**: Decision fatigue when starting a Python project — too many tool choices, too many config files to write.

| Design Element | How It Solves the Problem | Assessment |
|----------------|--------------------------|------------|
| Fixed toolchain (no alternatives) | Eliminates decision fatigue completely | **Strong** — opinionated is the point |
| Existing project detection (Step 1) | Avoids overwriting existing config | **Strong** — gap analysis pattern is practical |
| Incremental supplement (Step 2a) | Adds what's missing, doesn't break what works | **Strong** — respects existing projects |
| Keyword-based project type inference | Reduces questions to the user | **Good** — but limited to 3 types |
| Common configs embedded in SKILL.md | Agent has config templates ready, no lookup needed | **Good** — but duplicates content from reference files |
| Version placeholder for prek hooks | Forces query of latest versions at generation time | **Smart** — prevents stale hardcoded versions |

**Weaknesses**:
- **Only 3 project types** (library, web-api, data-pipeline). Common Python project types like CLI tool, Discord/Telegram bot, Jupyter notebook project, or Django app are not covered. The fallback ("present 3 types and ask") works but doesn't serve these users well.
- **No guidance on what to do AFTER scaffolding** beyond "run uv sync, run pytest". The skill should explicitly hand off to coach: "Your project is scaffolded. For ongoing development discipline, activate vibe-coding-coach."
- **Template content in reference files** — the SKILL.md says "Read the appropriate reference file" but doesn't include the templates inline. This is correct progressive disclosure, but means the agent needs an extra read step. If the reference files are large, this is fine; if they're small, inline would be faster.

## 4. Issues Summary

### Critical (blocks correct behavior)

None identified.

### High (degrades effectiveness)

| # | Skill | Issue | Recommendation |
|---|-------|-------|----------------|
| H1 | coach | No `when_to_use` field — trigger logic mixed into description | Extract trigger phrases into `when_to_use` for cleaner separation |
| H2 | scaffold | Missing `metadata` field — no version, author, or toolchain info | Add `metadata:` with version, author, toolchain for consistency |
| H3 | scaffold | No handoff to coach after scaffolding | Add explicit "next step: activate vibe-coding-coach" in After Generation section |

### Medium (suboptimal but functional)

| # | Skill | Issue | Recommendation |
|---|-------|-------|----------------|
| M1 | coach | Description ~750 chars, approaching 1536 combined limit | Trim or split into `description` + `when_to_use` |
| M2 | scaffold | Only 3 project types — CLI, bot, Django not covered | Consider a 4th type or explicit "custom" path |
| M3 | scaffold | No `license` field in frontmatter | Add `license: Apache-2.0` |
| M4 | both | No `paths` field — activates on any file type | Consider `paths: "**/*.py,pyproject.toml"` but current broad activation may be intentional for coach |
| M5 | coach | 449 lines is near the 500-line soft limit | Acceptable now, but monitor as features grow |
| M6 | scaffold | Common Configs section duplicates reference file content | Could reduce body size by keeping only ruff/mypy configs inline and moving prek/pytest to references |

### Low (cosmetic / future)

| # | Skill | Issue | Recommendation |
|---|-------|-------|----------------|
| L1 | coach | `paradigm` field in metadata is non-standard | Harmless, but adds no agent-facing value |
| L2 | scaffold | No `when_to_use` field | Could add explicit trigger phrases |
| L3 | both | No `allowed-tools` declaration | Consider declaring for transparency (experimental feature) |
| L4 | scaffold | SKILL.md body includes full prek config templates | Consider moving to references/ for token savings |

## 5. Recommendations

### Immediate (H-priority)

1. **Coach: Add `when_to_use` field** — Extract the 5 activation scenarios and trigger keywords from `description` into a separate `when_to_use` field. This follows Claude Code's intended separation: description = what, when_to_use = when.

2. **Scaffold: Add `metadata` block** — Add version, author, and toolchain info to match coach's level of metadata. This helps agents and skill managers reason about the skill.

3. **Scaffold: Add handoff to coach** — In the "After Generation" section, add: "For ongoing development with engineering discipline, activate vibe-coding-coach."

### Next Iteration (M-priority)

4. **Scaffold: Consider 4th project type** — A "custom" type that only generates pyproject.toml + toolchain configs without imposing directory structure, for projects that don't fit the 3 templates.

5. **Both: Consider `paths` field** — Test whether adding `paths: "**/*.py,pyproject.toml"` improves activation precision without missing important triggers.

6. **Coach: Monitor body size** — At 449 lines, it's healthy. If it grows past 500, consider extracting the "Python 反模式警示" and "工具映射" tables into a reference file.

### Long-term (L-priority)

7. **Coach: Consider `allowed-tools`** — Declare `Bash Read Edit Write Grep` for transparency.
8. **Both: Dynamic context injection** — Consider using `!`git status --short`` to inject current state at skill load time, giving the agent immediate context.
