# python-hunter Skill Suite Evaluation

> Date: 2026-05-14
> Perspective: Vibe coder using Python with AI-assisted coding

## Current Coverage

| Skill | Pain Points Addressed | Coverage Phase |
|-------|----------------------|----------------|
| **vibe-coding-coach** | Full-cycle engineering discipline: pre-generation risk assessment (Iron Rule 1), risk-proportional verification depth during generation (Iron Rule 4), post-generation completion verification (Iron Rules 2/3/5), security-sensitive code detection, escalation signals | **Entire coding cycle** |
| **vibe-coding-scaffold** | Project initialization, standardized toolchain, zero decision fatigue, 3 project types | Project setup |

Coach spans the complete coding cycle — it is not merely a post-generation verifier. Its five iron rules map to distinct phases:

- **Iron Rule 1** (understand first, then generate) → **Before** code generation: risk-graded understanding confirmation
- **Iron Rules 3 & 4** (verification structure, risk-proportional depth) → **During** generation: guides how deep to verify based on risk level
- **Iron Rules 2 & 5** (completion ≠ running, know when to escalate) → **After** generation: completion gates and human escalation signals

The gap is not in the cycle coverage itself, but in **specific domains** that coach intentionally leaves out of scope (debugging methodology, dependency hygiene, performance, deployment).

## Unaddressed Pain Points (Ranked by Impact)

### 1. Debugging AI-Generated Code — Biggest Gap

Research shows this is the **#1 pain point** for vibe coders. AI code is "90% there" — the remaining 10% involves subtle logic errors, hallucinated function calls, and off-by-one mistakes. Neither skill addresses debugging workflows.

> "It's not just fixing syntax errors. It's about untangling logic that's subtly flawed. It's about catching a hallucinated function call that looks plausible." — Santiago Valdarrama

**Suggested skill**: `vibe-coding-debugger` — Structured debugging workflow: hypothesis generation → minimal reproduction → binary search isolation → fix verification. Python-specific: traceback interpretation, `pdb`/`breakpoint()` workflow, async debugging patterns.

### 2. Dependency Hygiene / Supply Chain Security

A study of 2.2M AI-generated code samples found **440,000+ references to hallucinated package names**. 45% of AI-generated code contains high-risk security flaws. Coach's `pip-audit` only checks known CVEs, not hallucinated or non-existent packages.

**Suggested skill**: `vibe-coding-depcheck` — Hallucinated package detection, lock file integrity verification, dependency tree auditing, license compliance checking, PyPI typosquatting awareness.

### 3. Technical Debt Management / Prototype-to-Production Migration

64% of vibe coders report "instant success" followed by mounting technical debt. Coach's Prototype mode encourages speed, but the **migration path from Prototype to Production** lacks dedicated guidance.

**Suggested skill**: `vibe-coding-debttracker` — Code smell detection, technical debt quantification, incremental upgrade paths, Prototype-to-Production migration checklist, debt tracking over time.

### 4. Performance Optimization

AI prioritizes "works" over "works efficiently." Python-specific pitfalls (GIL, async misuse, memory leaks, N+1 queries) are common. No current skill addresses performance.

**Suggested skill**: `vibe-coding-perf` — Profiling workflow (`cProfile`/`py-spy`/`memray`), Python-specific bottleneck identification (GIL/async/memory), benchmark testing framework integration, hot path optimization patterns.

### 5. Test Quality Over Quantity

AI tends to generate many shallow tests that pass but don't catch real bugs. Coach's Iron Rule 3 establishes the principle ("verification structure > verification quantity") and defines what counts as a critical business rule. However, it does not provide concrete **test design methodology** — how to systematically derive high-value test cases from those rules.

**Suggested skill**: `vibe-coding-testsmith` — Test strategy design, boundary value analysis, property-based testing (Hypothesis), mutation testing integration, test pyramid guidance, "what NOT to test" heuristics. Would complement coach's Iron Rule 3 by providing the "how" behind the "what".

### 6. Deployment / DevOps

Scaffold only covers local development environment. The gap between `uv run pytest` and production deployment (Docker, CI/CD, cloud hosting) is unaddressed.

**Suggested skill**: `vibe-coding-shipper` — Dockerfile generation, CI/CD pipeline setup (GitHub Actions), cloud deployment patterns (AWS/GCP/Fly.io/Railway), environment variable management, health check and observability basics.

### 7. Context Management / Code Understanding

The "vibe coding paradox": developers using AI finish tasks faster but retain significantly less knowledge of what was built. No skill helps the developer understand and internalize AI-generated code.

**Suggested skill**: `vibe-coding-explainer` — On-demand code walkthrough generation, architecture documentation auto-generation, "explain this module" workflow, decision record keeping.

## Coverage Map

```
scaffold → coach (full cycle)            → gaps beyond coach's scope
  setup    ┌─────────────────────────┐
           │ Iron Rule 1: understand  │   → debug/optimize when code doesn't work
           │ Iron Rule 4: risk depth  │   → dependency hygiene
           │ Iron Rule 2: completion  │   → deployment / DevOps
           │ Iron Rule 3: test quality│   → tech debt tracking
           │ Iron Rule 5: escalation  │   → code understanding
           └─────────────────────────┘
```

Coach owns the engineering discipline loop. The gaps are **orthogonal domains** it deliberately defers — debugging methodology, dependency supply chain, performance tuning, deployment, and knowledge retention.

## Priority Recommendations

| Priority | Skill | Rationale |
|----------|-------|-----------|
| P0 | `vibe-coding-debugger` | #1 reported pain point across all vibe coding research |
| P0 | `vibe-coding-depcheck` | Security-critical; 440k+ hallucinated packages is a systemic risk |
| P1 | `vibe-coding-debttracker` | Technical debt is the long-term cost of vibe coding speed |
| P1 | `vibe-coding-testquality` | Directly extends coach's Iron Rule 3 with actionable guidance |
| P2 | `vibe-coding-perf` | Important but less frequent pain point |
| P2 | `vibe-coding-shipper` | Completes the end-to-end workflow |
| P3 | `vibe-coding-explainer` | Addresses knowledge retention but lower urgency |

## Sources

- [Vibe Coding in 2026: The Complete Guide](https://antigravity.codes/blog/vibe-coding-guide)
- [Vibe Coding in Practice: Motivations, Challenges, and a Future Outlook (arXiv)](https://arxiv.org/html/2510.00328v1)
- [Vibe Coding Statistics For 2026](https://subhrajyotimahato.com/blog/vibe-coding-statistics/) — 440k+ hallucinated packages, 45% high-risk security flaws
- [Stack Overflow: A new worst coder has entered the chat](https://stackoverflow.blog/2026/01/02/a-new-worst-coder-has-entered-the-chat-vibe-coding-without-code-knowledge/)
- [How to Debug with AI: 5 Tips](https://www.linkedin.com/posts/svpino_debugging-with-ai-is-an-a-experience-here-activity-7349096317597478915-pP3Y)
- [The State of Python 2025 (JetBrains)](https://blog.jetbrains.com/pycharm/2025/08/the-state-of-python-2025/)
