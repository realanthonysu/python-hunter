# python-hunter

> Python 专用的 AI 辅助编码工程纪律技能 — 因为"能跑 ≠ 完成"。

[English](./README.md) | 中文

![License](https://img.shields.io/badge/license-Apache--2.0-green)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)
![Python](https://img.shields.io/badge/Python-3.10+-3776ab)

一个兼容 [Agent Skills](https://agentskills.io/specification) 规范的 Python 专用技能套件，在 AI 辅助编码中强制执行 Python 特定的软件工程纪律。基于 Karpathy 的洞察：*思考可以外包，理解不能外包*。

这些技能最初是 [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian) 的一部分，现已提取到专门的 Python 仓库。

## 技能一览

| 技能 | 说明 | 适用范围 |
|------|------|----------|
| **vibe-coding-coach** | Python 工程纪律 — 将五条铁律映射到 ruff/mypy/pytest/bandit+semgrep 工具链 | Python 开发 |
| **vibe-coding-scaffold** | 项目脚手架 — 用 uv/ruff/mypy/ty/pytest/prek 标准化 Python 开发环境 | Python 项目初始化 |

`vibe-coding-coach` 定义*怎么在 Python 中验证*。`vibe-coding-scaffold` 确保验证基础设施从第一天就到位。

> 语言无关的核心技能（`vibe-coding-guardian`）仍在独立的 [仓库](https://github.com/realanthonysu/vibe-coding-guardian) 中。

---

## 亮点

### vibe-coding-coach (v1.1)

- **双运行模式** — 🚀 Prototype 追求速度（Jupyter 友好），🏭 Production 追求安全（PyPI 就绪）
- **Python 特有风险关键词** — 自动检测 `eval()`、`pickle.loads()`、`subprocess(shell=True)` 等
- **工具链自动探测** — 尊重项目已有的工具选择（ruff vs flake8，pytest vs unittest）
- **5 项可计算信号** — 用 `git diff` + `grep` 替代"直觉不安"

### vibe-coding-scaffold (v0.2)

- **零决策疲劳** — 一套固定工具链，无配置争论
- **3 种项目类型** — library（PyPI 包）、web-api（FastAPI 服务）、data-pipeline（数据管道）
- **增量补充** — 为已有项目添加缺失配置，不覆盖现有配置
- **Python 版本参数化** — 支持任意目标 Python 版本

---

## 目录

- [技能一览](#技能一览)
- [亮点](#亮点)
- [Python 工具链](#python-工具链)
- [前置条件](#前置条件)
- [安装](#安装)
- [使用方式](#使用方式)
- [文件结构](#文件结构)
- [相关技能](#相关技能)
- [版本演进](#版本演进)
- [许可证](#许可证)

---

## Python 工具链

两个技能共享一套固定的工具链：

| 职责 | 工具 | 为什么选它 |
|------|------|-----------|
| 包管理 | **uv** | 快速、Rust 实现、Python 包管理标准 |
| Lint + 格式化 | **ruff** | 替代 flake8 + isort + black，一个工具搞定 |
| 类型检查（深度） | **mypy** | 行业标准，捕获隐蔽的类型错误 |
| 类型检查（快速） | **ty** | Astral 新出的 Rust 实现检查器，即时反馈 |
| 测试 | **pytest** | 事实标准 |
| 覆盖率 | **coverage** | 测试完备性度量 |
| 安全扫描 | **bandit** + **semgrep** | Python 专用 SAST + OWASP Top 10 |
| Git hooks | **prek** | Rust 实现的 pre-commit 替代，更快更轻 |

---

## 前置条件

- **Claude Code**（最新版）、**Trae** 或任何兼容 **Agent Skills 规范**的 Agent
- **Python 3.10+**（coach）或 **Python 3.12+**（scaffold 默认）
- **Git**（可选 — 技能在没有 git 历史时也能工作，会自动降级）

---

## 安装

### Skill CLI（推荐）

```bash
# 安装 coach（Python 工程纪律）
npx skills add realanthonysu/vibe-coding-coach

# 安装 scaffold（Python 项目脚手架）
npx skills add realanthonysu/vibe-coding-scaffold
```

### Claude Code

**项目级**（跟随项目）：

```bash
mkdir -p .claude/skills
cp -r skills/vibe-coding-coach .claude/skills/
cp -r skills/vibe-coding-scaffold .claude/skills/
```

**用户级**（所有项目可用）：

```bash
mkdir -p ~/.claude/skills
cp -r skills/vibe-coding-coach ~/.claude/skills/
cp -r skills/vibe-coding-scaffold ~/.claude/skills/
```

### 其他 Agent

本技能遵循 [Agent Skills 规范](https://agentskills.io/specification)。将技能文件夹放到你的 Agent 技能目录中即可（如 `.agents/skills/`）。

---

## 使用方式

### vibe-coding-coach

无需手动激活 — 技能会在任何 Python 代码生成、修改或重构任务时自动激活。

```
你：帮我写一个 CSV 导出接口

AI：[自动激活 vibe-coding-coach]

🟡 中风险（新功能 + 文件 I/O）。需要确认几点：
1. 预计数据量 — 大数据集需要流式处理吗？
2. 编码 — UTF-8 还是需要 BOM 兼容 Excel？
3. 过滤 — 导出全部记录还是支持查询参数？

你：最多 10 万行，UTF-8，支持日期范围过滤

AI：[生成流式 CSV + Pydantic 查询参数 + 错误处理]
[ruff check ✅] [mypy ✅] [pytest ✅]
完成条件：✅ 正向路径 ✅ 负向路径 ✅ 无回归 ✅ 契约一致
```

### vibe-coding-scaffold

通过斜杠命令或自然语言触发：

```
/vibe-coding-scaffold

或

"帮我初始化一个 FastAPI 后端项目"
"Create a new Python library for data validation"
"给这个项目加上工程规范配置"
```

---

## 文件结构

```
python-hunter/
├── README.md                           # 英文说明
├── README_cn.md                        # 中文说明（本文件）
├── LICENSE                             # Apache-2.0
├── CHANGELOG.md                        # 版本变更记录
└── skills/
    ├── vibe-coding-coach/         # Python 工程纪律
    │   ├── SKILL.md                    # 核心技能（铁律 + 决策流程）
    │   ├── README.md                   # 英文说明
    │   └── references/                 # 按需加载的深度指南
    │       ├── architecture.md
    │       ├── security.md
    │       ├── quality-gates.md
    │       ├── refactoring.md
    │       └── examples.md
    └── vibe-coding-scaffold/             # Python 项目脚手架
        ├── SKILL.md                    # 核心技能（决策流程 + 模板）
        ├── README.md                   # 英文说明（含中文摘要）
        └── references/                 # 项目类型模板
            ├── library.md
            ├── web-api.md
            └── data-pipeline.md
```

---

## 版本演进

### vibe-coding-coach (v1.1)

| 版本 | 关键变更 |
|------|----------|
| v1.0.0 | 初版发布 — Python 版，将铁律映射到 ruff/mypy/ty/pytest/bandit/uv 工具链 |
| v1.1.0 | ty 降级为可选（Beta，typing spec 合规率 ~53%），安全扫描栈增加 Semgrep，新增 ML 反序列化风险关键词（torch.load/joblib.load），Gotchas 增加 Astral 收购风险提示 |

### vibe-coding-scaffold (v0.2)

| 版本 | 关键变更 |
|------|----------|
| v0.1.0 | 初版发布 — 固定工具链的 Python 项目脚手架（uv/ruff/mypy/ty/pytest/coverage/prek），支持 library/web-api/data-pipeline 三种项目类型 |
| v0.2.0 | Python 版本参数化，prek hook 版本号改为占位符（生成时查询最新版本），library 项目增加 py.typed 标记文件，Gotchas 增加 Astral 收购风险提示 |

详细变更记录见 [CHANGELOG.md](./CHANGELOG.md)。

---

## 许可证

[Apache-2.0](./LICENSE)
