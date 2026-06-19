---
title: W10 - CPython 源码导读
chapter: 01
week: 10
duration: 7 天 / 每天 2 小时
prerequisite: 完成 W01-W09，对 list/dict/字节码有基本了解，会一点 C
tags: [python, cpython, 源码, 字节码, 内存管理, gil]
---

# W10 · CPython 源码导读 🔬

> 本周目标：打开 CPython 的"引擎盖"。理解 list 为什么扩容是 1.125 倍、dict 怎么保持插入顺序、GIL 的代码长什么样、引用计数 + 分代 GC 怎么协作。最终写一篇 2000 字博客，解释 list 扩容策略。

---

## 1. CPython 是什么？🐍

你天天用的 `python`，其实是 **CPython** —— 用 C 语言写的 Python 解释器，由 Guido 等核心开发者维护。还有 PyPy（用 Python 写的、带 JIT）、Jython（跑在 JVM 上）、MicroPython 等等。本章只讲 CPython，因为它是事实标准。

> 💡 在终端敲：
> ```
> python3 -c "import platform; print(platform.python_implementation())"
> # CPython
> ```

---

## 2. 整体架构：从 .py 到机器执行 🛤️

```
你的 .py 源码
   │
   ▼
[Parser]  Parser/  把字符流切成 tokens、构建语法树
   │
   ▼
[AST]   抽象语法树（Abstract Syntax Tree）
   │
   ▼
[Compiler]  Python/compile.c  把 AST 编译成字节码
   │
   ▼
[Bytecode]  .pyc 文件里就是它
   │
   ▼
[Eval Loop]  Python/ceval.c  一条一条字节码执行（巨型 switch）
   │
   ▼
[C-level Object Ops]  PyListObject / PyDictObject / ...
```

**类比**：CPython 就是一台**专门为 Python 设计的虚拟机**。Java 有 JVM，Python 有 CPython VM。

---

## 3. 用 `dis` 看字节码：理解从这里开始 🔍

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

**输出**：
```
  2           0 RESUME                   0
  3           2 LOAD_FAST                0 (a)
              4 LOAD_FAST                1 (b)
              6 BINARY_OP                0 (+)
             10 RETURN_VALUE
```

| 字节码 | 含义 |
|-------|------|
| `RESUME` | 函数开始（3.11+ 引入） |
| `LOAD_FAST` | 把局部变量压入栈顶 |
| `BINARY_OP 0 (+)` | 弹出两个值、相加、压回栈 |
| `RETURN_VALUE` | 弹出栈顶作为返回值 |
| `CALL` | 调用函数 |
| `STORE_FAST` | 把栈顶存到局部变量 |
| `LOAD_GLOBAL` | 加载全局变量（比 LOAD_FAST 慢） |

> 💡 **划重点**：CPython 是**栈式虚拟机**——所有运算都在一个"操作数栈"上做：压栈、弹栈、运算、压栈。这是为什么 `LOAD_FAST` 比 `LOAD_GLOBAL` 快——前者直接数组下标，后者要查 dict。

### 🎯 动手练习 10.1
看下面三段代码的字节码，比较谁更快：

```python
def f1():
    return [x*2 for x in range(10)]

def f2():
    result = []
    for x in range(10):
        result.append(x*2)
    return result
```

<details>
<summary>👀 参考答案</summary>

```python
import dis
dis.dis(f1)
dis.dis(f2)
```

f1 用 `LIST_APPEND`（专门优化的字节码），f2 用 `LOAD_METHOD` + `CALL`。**f1 显著更快**，这是为什么列表推导比 for 循环快 30%。
</details>

---

## 4. 下载源码 + 目录速览 🗺️

```bash
git clone --depth 1 https://github.com/python/cpython.git
cd cpython
```

| 目录 | 内容 | 重要程度 |
|------|------|---------|
| `Include/` | 公开头文件 `.h`，定义 PyObject、PyListObject 等 | ⭐⭐⭐ |
| `Objects/` | 内置类型实现：list/dict/tuple/str/int 都在这里 | ⭐⭐⭐⭐⭐ |
| `Python/` | 解释器核心：ceval.c（执行循环）、import.c | ⭐⭐⭐⭐ |
| `Modules/` | 标准库 C 实现：os / itertools / _hashlib | ⭐⭐⭐ |
| `Lib/` | 纯 Python 写的标准库：collections / functools | ⭐⭐⭐⭐ |
| `Parser/` | 词法 + 语法分析 | ⭐⭐ |
| `Doc/` | 文档源码 | ⭐ |

> 💡 **入门最佳路线**：先读 `Objects/listobject.c` —— 里面没复杂宏，是最适合"第一次读 C 源码"的文件。

---

## 5. PyListObject：list 在 C 里长啥样 📚

`Include/cpython/listobject.h`：

```c
typedef struct {
    PyObject_VAR_HEAD          // 引用计数 + 类型指针 + 长度
    PyObject **ob_item;        // 实际元素指针数组（一组指向 PyObject 的指针）
    Py_ssize_t allocated;      // 已分配的空间（>= 实际长度）
} PyListObject;
```

**关键洞察**：list **不是**直接装值，而是装**指向 PyObject 的指针**。一个 `[1, "a", 3.14]` 在内存里是：

```
PyListObject
├── ob_size = 3
├── allocated = 4              ← 多分配 1 个空位，下次 append 不用扩容
└── ob_item ──► [ ptr1, ptr2, ptr3, _ ]
                  │     │     │
                  ▼     ▼     ▼
                PyLong "1"  PyUnicode "a"  PyFloat 3.14
```

这解释了：
- 为什么 list 能装混合类型？因为存的是**指针**。
- 为什么 list 越大越占内存？因为每个元素都要单独分配一个 PyObject。
- 为什么 NumPy 数组比 list 快？因为它是**连续的原始 double**，没指针、没 PyObject 头。

---

## 6. list 扩容策略：为什么是 1.125 倍 📈

`Objects/listobject.c` 的 `list_resize`：

```c
new_allocated = ((size_t)newsize + (newsize >> 3) + 6) & ~(size_t)3;
```

翻译成 Python：

```python
new_allocated = (newsize + newsize // 8 + 6) & ~3      # ~3 = 对齐到 4 的倍数
```

**展开**：每次扩容 `≈ newsize * 1.125 + 6`，再向上对齐到 4 的倍数。

| 当前长度 | 扩容后空间 |
|---------|----------|
| 0 → 1 | 4 |
| 4 → 5 | 8 |
| 8 → 9 | 16 |
| 16 → 17 | 24 |
| 32 → 33 | 40 |

**对比**：Java ArrayList 是 1.5 倍，C++ vector 是 2 倍。Python 选 1.125 倍是**省内存优先**——大量小列表加起来浪费空间会很可观。

> 💡 **均摊复杂度**：虽然单次扩容是 O(n)（要 memcpy），但每个元素被 copy 的总次数有上界，**均摊到每次 append 仍是 O(1)**。

### 实际测试

```python
import sys
lst = []
prev = sys.getsizeof(lst)
for i in range(40):
    lst.append(i)
    cur = sys.getsizeof(lst)
    if cur != prev:
        print(f"len={len(lst)} size={cur} bytes")
        prev = cur
```

---

## 7. PyDictObject：哈希表 + 保序 🔑

`Objects/dictobject.c` 是 CPython 最复杂、最优雅的代码之一。

### 哈希表基础回顾

| 冲突解决方式 | 例子 | CPython 用 |
|------------|------|-----------|
| **链地址法**（每个槽是个链表） | Java HashMap | ❌ |
| **开放寻址**（撞了就找下一个空位） | Python | ✅ |

CPython 用**伪随机探测**（`perturb` 变量），让冲突分布更均匀。

### PEP 468：dict 保持插入顺序

Python 3.7+ dict 默认有序。秘密在于**两层结构**：

```c
struct _dictkeysobject {
    Py_ssize_t dk_size;            // 哈希索引表
    Py_ssize_t dk_indices[];       // 紧凑索引：[0, 1, 2, ...]
    PyDictKeyEntry dk_entries[];   // 真正的 (key, hash, value) 数组，按插入顺序
};
```

**类比**：图书馆 = 书架 + 目录卡片。
- `dk_entries` = 书架（按入库时间摆放）
- `dk_indices` = 卡片（按字母索引指向书架位置）

冲突时只动卡片，不动书架，所以插入顺序保持。

### 性能关键：hash 函数

```python
print(hash("abc"))                # 整数
print(hash((1, 2, 3)))             # tuple 可哈希
# print(hash([1, 2]))              # 报错：list 不可哈希
```

> 💡 **划重点**：可变对象不可哈希——因为内容一改，hash 就变了，dict 就找不回来了。这是为什么 list 不能当 key、tuple 可以。

### 🎯 动手练习 10.2
解释为什么这段代码会出 bug：

```python
key = [1, 2, 3]
d = {tuple(key): "v"}
key.append(4)
print(d.get(tuple(key)))   # None
```

<details>
<summary>👀 参考答案</summary>

`tuple(key)` 第一次是 `(1,2,3)`，第二次是 `(1,2,3,4)`，hash 不同，自然找不到。**dict 的 key 在存进去那一刻就被"拍照"了，后续改 key 没用**。
</details>

---

## 8. ceval.c：字节码执行循环 🔄

`Python/ceval.c` 有 5000+ 行，核心是一个巨型 switch：

```c
// 简化版
for (;;) {
    opcode = NEXTOP();
    switch (opcode) {
        case LOAD_FAST: {
            value = GETLOCAL(oparg);
            PUSH(value);
            DISPATCH();
        }
        case BINARY_OP: {
            right = POP();
            left = POP();
            res = PyNumber_Add(left, right);
            PUSH(res);
            DISPATCH();
        }
        case RETURN_VALUE:
            return POP();
        // ... 100+ 种 opcode
    }
}
```

3.11 引入 **specializing adaptive interpreter**——遇到稳定的类型（比如总是 int + int），自动把 `BINARY_OP` 替换成更快的专用版本。这是 3.11 比 3.10 快 25% 的核心原因。

---

## 9. GIL：在源码里长这样 🔐

`Python/ceval_gil.c`：

```c
static int
take_gil(PyThreadState *tstate)
{
    MUTEX_LOCK(gil->mutex);
    while (_Py_atomic_load_relaxed(&gil->locked)) {
        // 自旋 + 条件变量等
        COND_TIMED_WAIT(gil->cond, gil->mutex, INTERVAL);
    }
    _Py_atomic_store_relaxed(&gil->locked, 1);
    MUTEX_UNLOCK(gil->mutex);
    return 1;
}
```

每跑约 100 条字节码或遇到 IO，会调 `drop_gil` 让出。

> 🚀 **重大新闻**：PEP 703（3.13+ 实验性 `--without-gil` 编译选项）正在推进**移除 GIL**。未来几年 Python 多线程会真正能跑满多核。

---

## 10. 内存管理：引用计数 + 分代 GC 🧹

### 引用计数（主力）

每个 PyObject 都有 `ob_refcnt` 字段：

```c
typedef struct _object {
    Py_ssize_t ob_refcnt;          // 引用计数
    PyTypeObject *ob_type;
} PyObject;
```

```python
import sys
a = []
print(sys.getrefcount(a))    # 2（a 自己 + getrefcount 临时引用）
b = a
print(sys.getrefcount(a))    # 3
del b
print(sys.getrefcount(a))    # 2
```

**原则**：refcnt = 0 立刻释放。

### 分代 GC（处理循环引用）

```python
a = []; b = []
a.append(b); b.append(a)     # a 和 b 互相引用，refcnt 永远 ≥ 1
del a, b                     # refcnt 降不到 0，永远不释放？
```

GC 模块定期扫描，找到这种**循环垃圾**回收掉。分**三代**（gen 0/1/2），新对象在 gen 0，活下来的升 gen。

```python
import gc
print(gc.get_count())        # (gen0_count, gen1, gen2)
gc.collect()                  # 手动触发
```

---

## 11. 实战：写一篇博客《Python list 的扩容策略与时间复杂度》📝

**任务**：2000 字 + 内存布局图。

### 推荐结构
1. **引子**（200 字）：从一段 `for i in range(N): lst.append(i)` 开始，问读者"这是 O(n) 还是 O(n²)？"
2. **list 内存布局**（400 字）：画 PyListObject 三字段图，说明指针数组。
3. **扩容公式与曲线**（500 字）：贴 `list_resize` 公式，给出 0→1→4→8→16…的表，画扩容曲线。
4. **均摊分析**（400 字）：证明每次 append 均摊 O(1)。
5. **对比 Java/C++**（300 字）：1.5 倍 / 2 倍 / 1.125 倍的取舍。
6. **实测代码**（200 字）：用 `sys.getsizeof` 实测画图。

### 内存布局图（ASCII 即可）

```
┌─────────────────────────────────────┐
│ PyListObject                        │
├─────────────────────────────────────┤
│ ob_refcnt = 1                       │
│ ob_type   = &PyList_Type            │
│ ob_size   = 3   (实际元素数)         │
│ allocated = 4   (已分配空间)         │
│ ob_item ─────► [p0, p1, p2, _ ]    │
└─────────────────────────────────────┘
                    │   │   │
                    ▼   ▼   ▼
                   42  "x" 3.14
                 (PyLong/PyUnicode/PyFloat)
```

### 🎯 验收清单
- [ ] 字数 ≥ 2000
- [ ] 至少 1 张内存布局图
- [ ] 至少 1 段 `sys.getsizeof` 实测代码 + 输出
- [ ] 引用了 `Objects/listobject.c` 的真实公式
- [ ] 解释了"为什么是 1.125 倍"

---

## ✅ 本周检查清单

- [ ] 能画出 CPython 的 5 阶段流程图
- [ ] 用 `dis` 读过至少 3 个函数的字节码
- [ ] 在本机克隆了 cpython 仓库
- [ ] 读完了 `Objects/listobject.c`（至少看懂 `list_resize`）
- [ ] 知道 dict 怎么保持插入顺序（双层结构）
- [ ] 能解释引用计数 vs 分代 GC 的分工
- [ ] 知道 GIL 在 `ceval_gil.c`
- [ ] 写完了 list 扩容博客 ≥ 2000 字

## 🐛 常见误区

| 误区 | 真相 |
|------|------|
| "GIL 让多线程毫无用处" | IO 密集场景多线程仍然有用 |
| "dict 是 O(1) 永远成立" | 极端 hash 冲突会退化到 O(n) |
| "list 越大越慢插末尾" | 末尾 append 均摊 O(1)，跟大小无关 |
| "del x 立即释放内存" | 仅 refcnt-1，归 0 才释放 |
| "Python 没编译" | 有！.pyc 就是字节码，只是没编译到机器码 |

## 📚 延伸阅读

- 《CPython Internals》Anthony Shaw —— **强烈推荐**，系统讲源码
- 《Python 源码剖析》陈儒 —— 中文经典（基于 2.5，思想仍受用）
- CPython 官方开发文档：https://devguide.python.org/
- Brandt Bucher - "Specializing the Python Interpreter"：https://www.youtube.com/watch?v=shQtrn1v7sQ
- PEP 659（自适应解释器）：https://peps.python.org/pep-0659/
- PEP 703（无 GIL）：https://peps.python.org/pep-0703/

---

## 🎓 第 01 章总结

恭喜你！你已经走过：

```
W01 语法 → W02 数据结构 → W03 函数 → W04 OOP → W05 装饰器
       → W06 生成器 → W07 并发 → W08 类型 → W09 工程化 → W10 源码
```

10 周下来，你不再是"用 Python 写脚本的人"，而是"理解 Python 怎么跑"的人。

> **下一站**：[[02-Web前端基础]] —— 把后端能力延伸到浏览器，让世界看到你写的东西。

🐍 **Python，到此为止。但你只是刚刚开始。**
