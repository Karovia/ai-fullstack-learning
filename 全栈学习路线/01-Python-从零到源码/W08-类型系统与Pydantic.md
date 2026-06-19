---
title: W08 - 类型系统与 Pydantic
chapter: 01
week: 8
duration: 7 天 / 每天 1.5 小时
prerequisite: 完成 W03（函数）+ W04（OOP / dataclass）
tags: [python, 类型注解, typing, mypy, pydantic, 工程化]
---

# W08 · 类型系统与 Pydantic 📐

> 本周目标：让你的代码"自带说明书"。读 100 行别人的代码，从看 5 分钟变看 30 秒；从"运行时炸了"变"写代码时 IDE 就标红"；学会用 **Pydantic** 优雅地校验输入数据。

---

## 1. 为什么需要类型注解？📜

**类比**：写代码 = 签合同。
- 没注解：合同上写"甲方给乙方东西"——给啥？什么时候？数量？全凭口头约定，扯皮起步。
- 有注解：合同上写"甲方在 2026/06/19 前给乙方 100 个苹果"——一目了然。

```python
# 没注解（传错了运行时才炸）
def add(a, b):
    return a + b

add("1", 2)        # 运行时 TypeError

# 有注解（IDE / mypy 提前告诉你错了）
def add(a: int, b: int) -> int:
    return a + b
```

> 💡 **划重点**：Python 是**动态类型**语言，注解**不会在运行时检查**（除非你用 Pydantic / typeguard）。它是给**人和工具**看的。

---

## 2. 基础类型注解 🧱

```python
name: str = "Alice"
age: int = 18
height: float = 1.75
is_student: bool = True

scores: list[int] = [90, 80, 70]                 # Python 3.9+ 直接用内置类型
profile: dict[str, str | int] = {"name": "A", "age": 18}
point: tuple[float, float] = (1.0, 2.0)

def greet(name: str, times: int = 1) -> None:
    for _ in range(times):
        print(f"hi {name}")
```

> ⚠️ Python 3.8 及之前要 `from typing import List, Dict, Tuple`，写 `List[int]`。3.9+ 推荐用小写内置 `list[int]`。**本教程默认 3.10+**。

---

## 3. typing 模块进阶 🛠️

| 注解 | 含义 | 例子 |
|------|------|------|
| `Optional[X]` 或 `X \| None` | X 或 None | `name: str \| None = None` |
| `Union[A, B]` 或 `A \| B` | A 或 B | `x: int \| str` |
| `Literal["a", "b"]` | 只能是这几个具体值 | `mode: Literal["r", "w"]` |
| `Any` | 任何类型（关掉检查） | `data: Any` |
| `Callable[[int, str], bool]` | 一个吃 (int, str) 返回 bool 的函数 | 见下 |
| `Final[int]` | 常量，不能再赋值 | `PI: Final = 3.14` |

```python
from typing import Literal, Callable

def open_file(path: str, mode: Literal["r", "w", "a"] = "r") -> None:
    print(path, mode)

open_file("a.txt", "x")    # mypy 报错：Literal 只允许 r/w/a

def apply(fn: Callable[[int], int], x: int) -> int:
    return fn(x)

print(apply(lambda n: n * 2, 5))      # 10
```

### 🎯 动手练习 8.1
写一个 `parse_age(value: str | int) -> int`，传字符串和整数都能解析，非数字字符串抛 `ValueError`。

<details>
<summary>👀 参考答案</summary>

```python
def parse_age(value: str | int) -> int:
    if isinstance(value, int):
        return value
    if value.isdigit():
        return int(value)
    raise ValueError(f"非法年龄：{value!r}")

print(parse_age(18), parse_age("20"))     # 18 20
```
</details>

---

## 4. 泛型 TypeVar：给容器一个"占位符" 📦

如果你写了一个"装任何类型的栈"，你不希望写两次（一次给 int 一次给 str）：

```python
from typing import TypeVar, Generic

T = TypeVar("T")                                 # T 是占位符

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, x: T) -> None:
        self._items.append(x)

    def pop(self) -> T:
        return self._items.pop()

s: Stack[int] = Stack()
s.push(1)
s.push("oops")     # mypy 报错：期望 int，得到 str
```

---

## 5. Protocol：结构化类型 = 鸭子类型 + 检查 🦆

**类比**：你不在乎它**叫什么名字**（class），只在乎它**会不会嘎嘎叫**（方法）。

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("画圆")

class Square:
    def draw(self) -> None:
        print("画方")

def render(items: list[Drawable]) -> None:
    for it in items:
        it.draw()

render([Circle(), Square()])     # ✅ 它们都没继承 Drawable，但都"长得像"
```

> 💡 Protocol 是 Python 类型系统里最优雅的设计之一，比 Java 的 interface 灵活。

---

## 6. TypedDict：给字典加结构 📋

```python
from typing import TypedDict

class User(TypedDict):
    id: int
    name: str
    email: str

u: User = {"id": 1, "name": "Alice", "email": "a@x.com"}    # ✅
bad: User = {"id": "1"}                                       # mypy 报错
```

适合处理 JSON / API 响应。

---

## 7. 类型别名：给复杂类型起小名 🏷️

```python
type UserId = int                       # Python 3.12+ 新语法
type Vector = list[float]
type JSON = dict[str, "JSON"] | list["JSON"] | str | int | float | bool | None

def add_one(uid: UserId) -> UserId:
    return uid + 1
```

---

## 8. mypy：静态类型检查 🔍

安装：

```bash
pip install mypy
mypy your_file.py
```

例子：

```python
# bad.py
def add(a: int, b: int) -> int:
    return a + b

print(add("1", 2))
```

```bash
$ mypy bad.py
bad.py:4: error: Argument 1 to "add" has incompatible type "str"; expected "int"
Found 1 error in 1 file
```

常用配置 `pyproject.toml`：

```toml
[tool.mypy]
python_version = "3.12"
strict = true                  # 全部最严格
disallow_untyped_defs = true   # 函数必须写注解
```

---

## 9. ruff：超快的代码风格 + 类型检查工具 ⚡

ruff 用 Rust 写，比 flake8 / black / isort 快 10-100 倍。**一个工具替代三个**。

```bash
pip install ruff
ruff check .          # 静态检查
ruff format .         # 自动格式化
```

`pyproject.toml`：

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]  # 错误/未用/import 排序/升级语法/bugbear/简化
```

> 💡 推荐 VSCode 装 "Ruff" 插件，保存自动格式化。

---

## 10. Pydantic v2：数据校验之王 🛡️

类比：dataclass 是"自动写 `__init__` 的傻瓜模式"，Pydantic 是"dataclass + 在运行时检查类型 + 自动转换 + 错误信息漂亮"。

安装：

```bash
pip install pydantic
```

```python
from pydantic import BaseModel, Field, EmailStr, field_validator

class User(BaseModel):
    id: int
    name: str = Field(min_length=1, max_length=20)
    age: int = Field(ge=0, le=150)        # ge=大于等于, le=小于等于
    email: EmailStr                        # 自动校验邮箱格式
    tags: list[str] = []

    @field_validator("name")
    @classmethod
    def name_no_space(cls, v: str) -> str:
        if " " in v:
            raise ValueError("name 不能含空格")
        return v.strip()

# 字符串自动转 int
u = User(id="1", name="Alice", age=18, email="a@x.com")
print(u)

# 校验失败
try:
    User(id="x", name="A B", age=-1, email="not-an-email")
except Exception as e:
    print(e)
```

**运行结果**：
```
id=1 name='Alice' age=18 email='a@x.com' tags=[]
4 validation errors for User
id
  Input should be a valid integer ...
name
  Value error, name 不能含空格 ...
age
  Input should be greater than or equal to 0 ...
email
  value is not a valid email address ...
```

### 序列化：model_dump / model_dump_json

```python
print(u.model_dump())              # 转 dict
print(u.model_dump_json(indent=2)) # 转 JSON
```

### 反序列化

```python
data = '{"id": 2, "name": "Bob", "age": 30, "email": "b@x.com"}'
u2 = User.model_validate_json(data)
print(u2)
```

---

## 11. 实战：用 Pydantic 重构 W3 的 cli-todo 🎯

W3 的 todo 是一个 list of dict，没校验，乱传值会出各种问题。我们用 Pydantic 重写：

```python
# todo.py
from pydantic import BaseModel, Field
from datetime import datetime
from pathlib import Path
from typing import Literal
import json

class Task(BaseModel):
    id: int
    title: str = Field(min_length=1, max_length=80)
    status: Literal["todo", "doing", "done"] = "todo"
    created_at: datetime = Field(default_factory=datetime.now)

class TaskList(BaseModel):
    tasks: list[Task] = []

    def add(self, title: str) -> Task:
        new_id = max((t.id for t in self.tasks), default=0) + 1
        t = Task(id=new_id, title=title)
        self.tasks.append(t)
        return t

    def done(self, tid: int) -> None:
        for t in self.tasks:
            if t.id == tid:
                t.status = "done"
                return
        raise ValueError(f"任务不存在 id={tid}")

    def save(self, path: Path) -> None:
        path.write_text(self.model_dump_json(indent=2), encoding="utf-8")

    @classmethod
    def load(cls, path: Path) -> "TaskList":
        if not path.exists():
            return cls()
        return cls.model_validate_json(path.read_text(encoding="utf-8"))

# 主程序
if __name__ == "__main__":
    db = Path("tasks.json")
    tl = TaskList.load(db)
    tl.add("写 W08 教程")
    tl.add("学 Pydantic")
    tl.done(1)
    tl.save(db)
    print(tl.model_dump_json(indent=2))
```

**运行结果**（`tasks.json`）：
```json
{
  "tasks": [
    {"id": 1, "title": "写 W08 教程", "status": "done", "created_at": "..."},
    {"id": 2, "title": "学 Pydantic", "status": "todo", "created_at": "..."}
  ]
}
```

### 🎯 动手练习 8.2
给 `Task` 增加字段 `priority: Literal["low", "mid", "high"] = "mid"` 和 `due: datetime | None = None`，并加一个校验器：截止时间不能早于创建时间。

<details>
<summary>👀 参考答案</summary>

```python
from pydantic import model_validator

class Task(BaseModel):
    id: int
    title: str
    priority: Literal["low", "mid", "high"] = "mid"
    due: datetime | None = None
    created_at: datetime = Field(default_factory=datetime.now)

    @model_validator(mode="after")
    def check_due(self):
        if self.due and self.due < self.created_at:
            raise ValueError("due 不能早于 created_at")
        return self
```
</details>

---

## ✅ 本周检查清单

- [ ] 能给函数参数和返回值加正确的类型注解
- [ ] 知道 `Optional[X]` = `X | None`
- [ ] 用过 `Literal` 限定字符串取值
- [ ] 写过一个用 `TypeVar` 的泛型类
- [ ] 写过一个 `Protocol` 接口
- [ ] 在项目里跑过 mypy 和 ruff
- [ ] 用 Pydantic 写过一个 `BaseModel` 并触发过校验错误
- [ ] 把 W03 的 cli-todo 用 Pydantic 重构了

## 🐛 常见错误集合

| 错误 | 原因 | 解决 |
|------|------|------|
| `name 'list' is not subscriptable` | Python < 3.9 | 用 `from __future__ import annotations` 或 `from typing import List` |
| `Optional[int]` 还能传 None | 注解只是给工具看 | 用 mypy 检查 / Pydantic 运行时校验 |
| Pydantic v1 用法在 v2 报错 | API 大改 | 看官方迁移指南：v1 `dict()` → v2 `model_dump()` |
| `EmailStr` ImportError | 没装 | `pip install "pydantic[email]"` |
| mypy 太严格、报老代码错 | 一开始就 strict | 渐进开严：先 `disallow_untyped_defs`，再加其他 |

## 📚 延伸阅读

- 官方 typing：https://docs.python.org/zh-cn/3/library/typing.html
- mypy 官方文档：https://mypy.readthedocs.io/
- Pydantic v2 文档：https://docs.pydantic.dev/
- ruff 文档：https://docs.astral.sh/ruff/
- PEP 484（类型注解）：https://peps.python.org/pep-0484/
- PEP 544（Protocol）：https://peps.python.org/pep-0544/

> 🎉 学完本章，你的代码已经"自带说明书"。下周我们把它**打包发布到 PyPI**，让全世界 `pip install` 你的工具。
