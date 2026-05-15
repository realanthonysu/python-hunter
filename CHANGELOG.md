# Changelog

> Python-specific skills extracted from [vibe-coding-guardian](https://github.com/realanthonysu/vibe-coding-guardian).


## vibe-coding-coach v1.1.0

- ty 降级为可选（Beta）— 标注 typing spec 合规率 ~53%，建议作为 mypy 补充而非替代
- 安全扫描栈增加 Semgrep — 推荐 bandit + semgrep 组合使用，覆盖 OWASP Top 10
- 高风险关键词增加 torch.load() / joblib.load() / shelve.open()（ML 反序列化风险）
- security.md 新增"ML 模型反序列化"防护章节（weights_only=True / safetensors）
- Gotchas 增加 Astral 被 OpenAI 收购的生态风险提示

## vibe-coding-scaffold v0.2.0

- Python 版本参数化 — 不再硬编码 3.12，支持用户指定目标版本（默认 3.12）
- prek hook 版本号改为占位符 — 生成时必须查询最新版本，禁止使用过时硬编码版本
- library 项目类型增加 py.typed 标记文件（PEP 561），hatchling 配置同步更新
- Gotchas 增加 Astral 被 OpenAI 收购的生态风险提示

## vibe-coding-coach v1.0.0

- 初版发布 — Python 版，将铁律映射到 ruff/mypy/ty/pytest/bandit/uv 工具链

## vibe-coding-scaffold v0.1.0

- 初版发布 — 固定工具链的 Python 项目脚手架（uv/ruff/mypy/ty/pytest/coverage/prek），支持 library/web-api/data-pipeline 三种项目类型
