# Async 安全指南（Python）

加载条件：触及 asyncio 并发、多线程/多进程、共享可变状态、引入 async/await 代码

---

## 核心原则

**Async 不是魔法，是显式的协作式并发。** 事件循环是单线程的，阻塞调用会卡住整个循环。

---

## 事件循环阻塞检测

### 禁止在 async 函数中使用的阻塞调用

| 阻塞调用 | 问题 | 替代方案 |
|----------|------|----------|
| `time.sleep()` | 阻塞整个事件循环 | `await asyncio.sleep()` |
| `requests.get()` | 同步 HTTP 阻塞 | `aiohttp` / `httpx.AsyncClient` |
| `subprocess.run()` | 同步子进程阻塞 | `asyncio.create_subprocess_exec()` |
| `os.system()` | 同步 shell 阻塞 | `asyncio.create_subprocess_shell()` |
| `socket.recv()` | 同步 socket 阻塞 | `aio` 库（aiofiles, aioredis） |
| 密集 CPU 计算 | 阻塞事件循环 | `loop.run_in_executor()` 或 `asyncio.to_thread()` |

### 检测脚本

```bash
# 扫描 async 函数中的阻塞调用
grep -rn "async def" src/ | while read line; do
    file=$(echo "$line" | cut -d: -f1)
    lineno=$(echo "$line" | cut -d: -f2)
    # 检查函数体内是否有 time.sleep, requests.get 等
    ...
done
```

或使用 bandit 插件：
```bash
bandit -r src/ -r --skip B101  # B101 是 assert，跳过
```

---

## 任务生命周期管理

### ✅ Good — 任务创建与回收

```python
import asyncio
from typing import Coroutine, Any


class TaskManager:
    """管理 asyncio task 的生命周期。"""

    def __init__(self) -> None:
        self._tasks: set[asyncio.Task[Any]] = set()

    def spawn(self, coro: Coroutine[Any, Any, Any]) -> asyncio.Task[Any]:
        """创建任务并跟踪引用。"""
        task = asyncio.create_task(coro)
        self._tasks.add(task)
        task.add_done_callback(self._tasks.discard)  # 完成后自动清理
        return task

    async def shutdown(self) -> None:
        """优雅关闭：取消所有运行中的任务。"""
        for task in self._tasks:
            task.cancel()
        await asyncio.gather(*self._tasks, return_exceptions=True)
```

### ❌ Bad — 任务泄漏

```python
# ❌ 创建了任务但没有引用，无法取消或监控
async def background_worker():
    while True:
        await process_queue()
        await asyncio.sleep(1)

async def main():
    asyncio.create_task(background_worker())  # 🔥 任务泄漏！
    # 没有引用，main 结束后任务仍在运行
```

---

## 任务取消的正确处理

### ✅ Good — 响应 CancelledError

```python
import asyncio


async def long_running_operation() -> str:
    try:
        # 模拟长时间工作
        for i in range(10):
            await asyncio.sleep(1)
            print(f"Step {i + 1}/10")
        return "done"
    except asyncio.CancelledError:
        # ✅ 清理资源、关闭连接、回滚事务
        print("Operation cancelled, cleaning up...")
        raise  # ⚠️ 必须重新抛出 CancelledError
    finally:
        # finally 块总会执行，适合清理
        print("Cleanup complete")


async def main():
    task = asyncio.create_task(long_running_operation())
    await asyncio.sleep(3)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task was cancelled successfully")
```

### ❌ Bad — 吞掉 CancelledError

```python
async def bad_operation():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        pass  # 🔥 吞掉 CancelledError，任务无法正常取消
    return "done?"  # 继续执行，破坏取消语义
```

**规则**：捕获 `CancelledError` 后必须重新 `raise`。仅在 `finally` 中做清理。

---

## 共享状态与并发安全

### asyncio 锁

```python
import asyncio


class SharedCounter:
    """asyncio 安全的计数器。"""

    def __init__(self) -> None:
        self._count: int = 0
        self._lock = asyncio.Lock()

    async def increment(self) -> None:
        async with self._lock:
            current = self._count
            await asyncio.sleep(0)  # 模拟异步操作
            self._count = current + 1

    @property
    def count(self) -> int:
        return self._count
```

### ⚠️ 常见陷阱：asyncio Lock 与死锁

```python
# ❌ 不要在持有锁时 await 外部协程（可能死锁）
async def dangerous_pattern():
    async with lock:
        await external_service.call()  # 🔥 如果 external_service 也需要这个锁 → 死锁
```

**规则**：持有锁时只做本地状态变更，不在锁内 await 外部依赖。

---

## async with 和 async for

### Context Manager

```python
# ✅ 使用 async with 管理异步资源
import aiohttp


async def fetch(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
    # 连接自动关闭
```

### Iterator

```python
# ✅ 使用 async for 处理异步流
async def process_stream(queue: asyncio.Queue[str]) -> None:
    async for item in queue:
        await process(item)
```

---

## asyncio.gather vs asyncio.TaskGroup

### Python 3.11+ TaskGroup（推荐）

```python
async def fetch_all(urls: list[str]) -> list[str]:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(url)) for url in urls]
    # TaskGroup 保证：
    # 1. 所有任务完成后退出
    # 2. 任一任务异常，取消其余任务
    return [t.result() for t in tasks]
```

### Python 3.10 及以下 gather

```python
async def fetch_all_legacy(urls: list[str]) -> list[str]:
    results = await asyncio.gather(
        *[fetch(url) for url in urls],
        return_exceptions=True,  # ✅ 一个失败不取消其余
    )
    return [r for r in results if isinstance(r, str)]
```

---

## 超时控制

```python
import asyncio


async def fetch_with_timeout(url: str, timeout: float = 5.0) -> str:
    try:
        return await asyncio.wait_for(fetch(url), timeout=timeout)
    except asyncio.TimeoutError:
        raise ConnectionError(f"Request to {url} timed out after {timeout}s")
```

**规则**：所有外部 I/O 操作必须有超时，防止任务永远挂起。

---

## 并发模型选择

| 场景 | 推荐模型 | 原因 |
|------|----------|------|
| I/O 密集型（HTTP、DB、文件） | `asyncio` | 协程切换轻量，适合大量并发连接 |
| CPU 密集型（计算、图像处理） | `concurrent.futures.ProcessPoolExecutor` | 绕过 GIL，利用多核 |
| 混合 I/O + CPU | `asyncio` + `loop.run_in_executor()` | I/O 走事件循环，CPU 卸载到线程池 |
| 需要操作系统级别并发 | `multiprocessing` | 进程级隔离，适合不可信代码 |

### asyncio 卸载 CPU 任务

```python
import asyncio


def heavy_computation(data: bytes) -> str:
    # 这是同步函数，在普通线程中运行
    return process(data)


async def async_compute(data: bytes) -> str:
    loop = asyncio.get_running_loop()
    # ✅ 不阻塞事件循环，在线程池中执行
    result = await loop.run_in_executor(None, heavy_computation, data)
    return result
```

---

## async 测试

```python
import pytest


@pytest.mark.asyncio
async def test_async_fetch():
    result = await fetch("https://example.com")
    assert "Example Domain" in result


@pytest.mark.asyncio
async def test_concurrent_fetches():
    urls = ["https://example.com"] * 10
    results = await fetch_all(urls)
    assert len(results) == 10
```

需要 `pytest-asyncio`：
```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"  # 自动检测 async 测试
```

---

## 安全检查清单

- [ ] async 函数中没有阻塞调用（time.sleep, requests, 密集 CPU）？
- [ ] 所有创建的任务都有引用（不泄漏）？
- [ ] CancelledError 被正确重新抛出？
- [ ] 共享状态使用 asyncio.Lock 保护？
- [ ] 不在持有锁时 await 外部依赖？
- [ ] 所有外部 I/O 有超时控制？
- [ ] 使用 `async with` 管理异步资源？
- [ ] 任务在 shutdown 时被优雅取消？
- [ ] 测试使用 `pytest-asyncio` 标记？
