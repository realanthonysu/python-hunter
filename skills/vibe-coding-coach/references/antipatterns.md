# 反模式警示与 Gotchas（Python）

加载条件：代码评审、Vibe Coding 风险提示、完成条件漂移、测试策略讨论

---

## Python 反模式警示

| 反模式 | 表现 | 解药 |
|--------|------|------|
| 🚦 绿灯幻觉 | 测试全过，系统已坏 | 验证结构，不只看数量（铁律三） |
| 🔧 局部修补 | 修症状不修原因 | 追问"为什么"到根因（铁律一） |
| 🧠 上下文丢失 | 忘记为什么这样做 | 关键决策写进代码或文档 |
| 📦 依赖爆炸 | 随意 pip install | 评估必要性、维护状态、替代方案 |
| 💧 抽象泄漏 | 绕过封装走捷径 | 遵守层间契约，不跨层访问 |
| 🐍 Python 特有：可变默认参数 | `def f(x=[])` | 使用 `None` 哨兵：`def f(x=None)` |
| 🐍 Python 特有：裸 except | `except:` 捕获一切 | `except Exception:` 或更具体的异常类型 |
| 🐍 Python 特有：循环引用 | 模块间互相 import | 提取公共模块或使用延迟导入 |
| 🐍 Python 特有：未关闭资源 | 文件/连接未用 `with` | 始终使用 context manager (`with` 语句) |
| 🐍 Python 特有：类型注解缺失 | 公开 API 无类型提示 | 公开函数必须有完整类型注解 |

---

## Gotchas

### 验证与完成

- "应该已经测过了" 不等于 VERIFIED。没有证据的验证等于未验证。
- 覆盖率是输入不是结论。负向路径、契约漂移、跨层不变量它都无法回答。
- Vibe Coding 的风险不在代码不够快，而在"完成条件"开始漂移。
- 局部信号增长 ≠ 系统整体可信度增长。ruff 通过、mypy 通过、覆盖率没掉，不代表"完成"。
- 不是所有风险都适合继续自动化下沉。有些改动最合理的动作是拉人进来。

### 模式相关

- 🚀 Prototype 模式不等于"没有质量要求"——ruff 和 mypy 仍然必须通过。
- 原型代码有变成生产代码的倾向。切换到 🏭 Production 时，务必对已有代码做回溯审查。
- 关联测试 = 与改动文件直接相关的测试（通过文件路径或模块名匹配）。不是全量测试，也不是跳过测试。
- ⚠️ 已知风险：Prototype 无测试套件时的"人工确认核心行为"是弱约束——AI 无法验证用户声称的验证是否真实。这是速度与安全的折中，切换到 🏭 Production 时应补全测试。

### Python 生态

- Python 的动态特性是双刃剑：灵活但也容易隐藏错误。mypy --strict 能捕获大部分问题，但 duck typing 的边界仍需测试覆盖。
- ty 仍为 Beta 状态（typing spec 合规率 ~53%），速度极快但类型推断能力仍在追赶 mypy/pyright。建议作为 mypy 的快速反馈补充，不建议单独依赖 ty 做类型检查。
- `__init__.py` 的设计影响整个包的 API 表面——谨慎选择导出内容。

### 生态风险

- ⚠️ Astral（uv、ruff、ty 的母公司）于 2026 年 3 月被 OpenAI 收购。当前所有工具保持 MIT 开源许可，短期无影响。但长期存在与 OpenAI 生态深度绑定的风险。建议关注替代方案（Poetry/PDM 替代 uv，flake8+black 替代 ruff，pyright 替代 ty），并在关键项目中保持工具链的可替换性。

---

## 常见 Vibe Coding 翻车场景

### 场景 1：内存爆炸

```
用户：加个导出 CSV 功能
AI：（直接生成）→ 能跑了
用户：加个导入功能
AI：（直接生成）→ 能跑了
用户：加个定时任务自动导出
AI：（直接生成）→ 能跑了
用户：为什么生产环境内存爆了？
```

**根因**：每一步都是"能跑就行"，没有考虑编码一致性、大文件内存占用、错误处理、并发安全。局部绿灯持续累积，全局熵增到爆。

**解药**：每一步都评估风险、验证完成条件、处理边界情况。速度略慢，但不会积累隐性债务。

### 场景 2：测试数量幻觉

```python
def test_user_model():
    user = User(name="Alice", email="alice@test.com")
    assert user.name == "Alice"
    assert user.email == "alice@test.com"
    assert str(user) != ""
    assert user.id is not None
    assert user.created_at is not None
    assert user.updated_at is not None
    assert user.name != ""
    assert user.email != ""
    assert isinstance(user.id, int)
    assert isinstance(user.created_at, datetime)
    # ... 20 more trivial assertions
```

**根因**：10 个测试全过，但核心业务规则（邮箱唯一性、密码策略、验证流程）一个都没验证。

**解药**：用 **"关键行为是否可被验证"** 衡量信心，不用测试数量。

### 场景 3：绿灯幻觉

```
AI: ruff check ✅ mypy ✅ pytest ✅ 覆盖率 85%
AI: 完成！
```

**根因**：工具全通过，但负向路径（错误输入、异常状态）一个都没覆盖。

**解药**：完成 = 正向路径 + 负向路径 + 回归风险 + 契约一致。工具通过只是门槛，不是完成条件。
