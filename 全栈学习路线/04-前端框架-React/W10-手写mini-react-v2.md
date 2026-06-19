---
chapter: 04
week: 10
duration: 7天
tags: [react, fiber, hooks, useState, useEffect, diff, lane, concurrent, mini-react]
---

# W10 · 手写 mini-react v2（Hooks + Diff + Lane）

> 上周把"骨架"立起来了：Element → Fiber → Commit。
> 这周往里塞**灵魂**：useState、useEffect、diff、优先级调度。
> 写完你不仅会用 React，还能讲源码。

---

## 一、第 7 步：实现 useState

### 关键事实
1. 每个**函数组件**对应一个 fiber，fiber 上挂一个 `hooks: Hook[]` 数组。
2. 调用 `useState` 的**顺序**就是它在数组里的下标——所以 React 严禁条件调用 hook。
3. 每次组件 rerender，hooks 数组重新建，但通过 `alternate` 指向上一次的 fiber，把旧值拿过来。

### 数据结构

```ts
type Hook<T = any> = {
  state: T
  queue: Array<(prev: T) => T>   // 待消费的 setState
}

let wipFiber: Fiber | null = null
let hookIndex = 0
```

### 处理函数组件

```ts
function performUnitOfWork(fiber: Fiber): Fiber | null {
  if (typeof fiber.type === 'function') {
    updateFunctionComponent(fiber)
  } else {
    updateHostComponent(fiber)
  }
  // ...child/sibling/parent 同上周
}

function updateFunctionComponent(fiber: Fiber) {
  wipFiber = fiber
  hookIndex = 0
  wipFiber.hooks = []
  const children = [(fiber.type as Function)(fiber.props)]   // 调一次组件函数
  reconcileChildren(fiber, children)
}
```

### useState 本体

```ts
export function useState<T>(initial: T): [T, (a: T | ((p: T) => T)) => void] {
  const oldHook = wipFiber!.alternate?.hooks?.[hookIndex] as Hook<T> | undefined
  const hook: Hook<T> = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  }

  // 消费上一次累计的 setState
  const actions = oldHook?.queue ?? []
  actions.forEach(a => { hook.state = a(hook.state) })

  const setState = (action: T | ((p: T) => T)) => {
    const fn = (typeof action === 'function' ? action : (() => action)) as (p: T) => T
    hook.queue.push(fn)
    // 触发重新渲染：从根上重建 wip 树
    wipRoot = {
      ...currentRoot!,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }

  wipFiber!.hooks!.push(hook)
  hookIndex++
  return [hook.state, setState]
}
```

**这段是全周最重要的代码**，对照着白板画一画：
- 第一次渲染：`oldHook` 不存在 → 用 `initial`。
- 之后渲染：从 `alternate.hooks[i]` 拿到旧 state，把累积的 actions 消费掉。
- `setState` 只是把 action 推到 queue，然后**重置 wipRoot 让 workLoop 跑下一轮**。批量更新自然就实现了——一次事件里多次 setState，actions 都进同一个 queue，只触发一次 rerender。

---

## 二、第 8 步：实现 useEffect

```ts
type EffectHook = {
  tag: 'effect'
  create: () => (() => void) | void
  destroy?: () => void
  deps?: any[]
  next?: EffectHook
}
```

简化版（v2 教学版）：

```ts
export function useEffect(create: () => any, deps?: any[]) {
  const oldHook = wipFiber!.alternate?.hooks?.[hookIndex] as any
  const changed = !oldHook
    || !deps
    || deps.some((d, i) => !Object.is(d, oldHook.deps[i]))

  const hook = { tag: 'effect', create, deps, destroy: oldHook?.destroy }
  if (changed) wipFiber!.effects = [...(wipFiber!.effects ?? []), hook]
  wipFiber!.hooks!.push(hook)
  hookIndex++
}
```

在 `commitRoot` 之后跑：

```ts
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot!.child)
  // 跑 effects
  runEffects(wipRoot!)
  currentRoot = wipRoot
  wipRoot = null
}

function runEffects(fiber: Fiber | null) {
  if (!fiber) return
  fiber.effects?.forEach(e => {
    e.destroy?.()                  // 先 cleanup 上一次
    e.destroy = e.create()         // 跑这一次，记下 cleanup
  })
  runEffects(fiber.child)
  runEffects(fiber.sibling)
}
```

> 真实 React 把 effect 放到下一个浏览器空闲后异步跑（passive effect），我们这里同步跑只为教学。

---

## 三、第 9 步：Diff 算法

上周 reconcileChildren 一律标 PLACEMENT——更新和删除都没处理。补上。

```ts
function reconcileChildren(wipFiber: Fiber, elements: Element[]) {
  let oldFiber: Fiber | null = wipFiber.alternate?.child ?? null
  let prevSibling: Fiber | null = null
  let i = 0

  while (i < elements.length || oldFiber) {
    const el = elements[i]
    let newFiber: Fiber | null = null
    const sameType = !!oldFiber && !!el && oldFiber.type === el.type

    if (sameType) {                          // UPDATE：复用 dom，换 props
      newFiber = {
        type: oldFiber!.type,
        props: el.props,
        dom: oldFiber!.dom,
        parent: wipFiber,
        alternate: oldFiber!,
        child: null, sibling: null,
        effectTag: 'UPDATE',
      }
    } else if (el) {                         // PLACEMENT：新增
      newFiber = {
        type: el.type, props: el.props, dom: null, parent: wipFiber,
        alternate: null, child: null, sibling: null, effectTag: 'PLACEMENT',
      }
    }
    if (oldFiber && !sameType) {             // DELETION：删除
      oldFiber.effectTag = 'DELETION'
      deletions.push(oldFiber)
    }
    if (oldFiber) oldFiber = oldFiber.sibling

    if (i === 0) wipFiber.child = newFiber
    else if (newFiber) prevSibling!.sibling = newFiber
    if (newFiber) prevSibling = newFiber
    i++
  }
}
```

### 多节点 + key

上面是按下标 diff，**列表顺序变就翻车**：把第 3 项移到第 1 项 → 全部 unmount + 重建。
真实 React 用 **key**：先按 key 建 map，再按新顺序找——能复用就移动 (`Placement`)，不能复用才删。

教学版思路（伪代码）：
```ts
const oldMap = new Map<string, Fiber>()
let cur = oldFiber
while (cur) { oldMap.set(cur.props.key ?? cur.index, cur); cur = cur.sibling }
elements.forEach((el, i) => {
  const old = oldMap.get(el.props.key ?? i)
  if (old && old.type === el.type) {
    // 复用，可能移动
  } else {
    // 新建
  }
})
// 剩下没匹配到的 oldMap 全部 DELETION
```

### commit 增加 UPDATE / DELETION 分支

```ts
function commitWork(fiber: Fiber | null) {
  if (!fiber) return
  let p = fiber.parent
  while (p && !p.dom) p = p.parent
  const parentDom = p!.dom!

  if (fiber.effectTag === 'PLACEMENT' && fiber.dom) {
    parentDom.appendChild(fiber.dom)
  } else if (fiber.effectTag === 'UPDATE' && fiber.dom) {
    updateDom(fiber.dom, fiber.alternate!.props, fiber.props)
  } else if (fiber.effectTag === 'DELETION') {
    commitDeletion(fiber, parentDom); return
  }
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}

function commitDeletion(fiber: Fiber, parentDom: Node) {
  if (fiber.dom) parentDom.removeChild(fiber.dom)
  else commitDeletion(fiber.child!, parentDom)         // 函数组件没 dom，下钻
}

function updateDom(dom: any, prev: Props, next: Props) {
  // 1. 删旧事件 / 旧属性
  Object.keys(prev).filter(k => k !== 'children').forEach(name => {
    if (name.startsWith('on')) {
      dom.removeEventListener(name.toLowerCase().slice(2), prev[name])
    } else if (!(name in next)) {
      dom[name] = ''
    }
  })
  // 2. 加新事件 / 新属性
  Object.keys(next).filter(k => k !== 'children').forEach(name => {
    if (name.startsWith('on')) {
      dom.addEventListener(name.toLowerCase().slice(2), next[name])
    } else {
      dom[name] = next[name]
    }
  })
}
```

---

## 四、第 10 步：Lane 模型（优先级调度）

> 用户输入比 setTimeout 重要，过渡动画又比埋点重要——React 用 **Lane（车道）** 表达优先级。

| Lane | 二进制位 | 用途 |
| --- | --- | --- |
| `SyncLane` | `0b0001` | 离散事件（点击、输入），最高优先级 |
| `DefaultLane` | `0b0010` | `setState` 默认 |
| `TransitionLane` | `0b0100` | `startTransition`，可被打断 |
| `IdleLane` | `0b1000` | 闲时再做（埋点等） |

为什么用位图？因为一次更新可能**同时属于多条 lane**（比如同步事件里又触发了过渡），用位运算可以飞快地合并 / 提取。

```ts
const SyncLane = 0b0001
const DefaultLane = 0b0010
const TransitionLane = 0b0100

function pickHighestLane(lanes: number) {
  return lanes & (-lanes)            // 取最低位的 1
}
```

教学版只演示概念：

```ts
let pendingLanes = 0
function scheduleUpdate(lane: number) {
  pendingLanes |= lane
  const lane2run = pickHighestLane(pendingLanes)
  if (lane2run === SyncLane) {
    flushSync()                       // 立即跑
  } else {
    requestIdleCallback(workLoop)     // 让 workLoop 跑
  }
}
```

> 真实 React 在 `markRootUpdated` / `getNextLanes` 里做了非常细的位运算，配合 expirationTime 兜底"饥饿"。看 big-react 的 `react-reconciler/src/fiberLanes.ts` 是最佳路径。

---

## 五、第 11 步：并发渲染（可中断）

我们已经写过 `workLoop` + `deadline.timeRemaining()`，所以**已经是并发的**。要让它"看起来更并发"，可加：

- `useTransition` / `useDeferredValue` 把"非紧急更新"标到 TransitionLane。
- workLoop 中遇到更高优先级 update，**丢掉当前 wip 树**，从 currentRoot 重建。
- commit 阶段不可中断（保持 DOM 一致性）。

教学版可加一个粗糙 `startTransition`：

```ts
let currentLane = DefaultLane
export function startTransition(fn: () => void) {
  const prev = currentLane
  currentLane = TransitionLane
  try { fn() } finally { currentLane = prev }
}
// useState 的 setState 内：scheduleUpdate(currentLane)
```

---

## 六、第 12 步：三类 Fiber

| Tag | 什么 | dom 字段 | 怎么处理 |
| --- | --- | --- | --- |
| `HostRoot` | 整棵树根 (`render` 入口) | 容器 dom | 不创建新 dom，children 挂到容器 |
| `HostComponent` | `'div'` `'span'` 等字符串 | 自己创建的 dom | createDom + updateDom |
| `FunctionComponent` | `function App(){}` | null | 调用函数拿 children，自身没有 dom |

`commitWork` 找 `parentDom` 时要**跳过函数组件**——上面代码已处理。

---

## 七、项目验收：跑一个小应用

目标：用 mini-react 写下面应用并跑通。

```tsx
/** @jsx MiniReact.createElement */
import MiniReact, { useState, useEffect } from './mini-react'

function TodoApp() {
  const [list, setList] = useState<string[]>([])
  const [text, setText] = useState('')

  useEffect(() => {
    console.log('list 变了', list)
    return () => console.log('cleanup')
  }, [list])

  return (
    <div>
      <input value={text} onInput={(e: any) => setText(e.target.value)} />
      <button onClick={() => { setList([...list, text]); setText('') }}>
        添加
      </button>
      {list.length === 0 ? <p>空空如也</p> :
        <ul>
          {list.map((t, i) => (
            <li key={i} onClick={() => setList(list.filter((_, j) => j !== i))}>
              {t}
            </li>
          ))}
        </ul>
      }
    </div>
  )
}

MiniReact.render(<TodoApp />, document.getElementById('app')!)
```

通过的标志：
- 输入字符不丢失（useState 工作）。
- 添加 / 删除项后 DOM 正确（diff + UPDATE/DELETION 工作）。
- 切换"空 / 列表"分支不报错（条件渲染）。
- console 看到 effect 的 log（useEffect 工作）。

---

## 八、写一篇博客《React Fiber 是什么，为什么需要它》

要点提示（**一定要带图**）：
1. 旧 stack reconciler 的卡顿现象——递归不可中断。
2. 把递归改成链表 → 工作单元化。
3. 工作单元 + `requestIdleCallback` → 时间切片。
4. 双缓冲保证一致性。
5. Lane 模型表达优先级，配合并发模式。
6. 我自己写的 mini-react 跑通了 demo，附 GitHub 链接。

---

## 周末 Checklist

- [ ] useState 跑通（多次调用顺序对得上）
- [ ] useEffect 跑通（依赖数组 + cleanup）
- [ ] 三种 effectTag (PLACEMENT / UPDATE / DELETION) 都验证过
- [ ] 列表加 key，故意打乱顺序仍然 OK
- [ ] 能讲清楚 Lane 模型为什么用位图
- [ ] 区分 HostRoot / HostComponent / FunctionComponent
- [ ] TodoApp demo 完整跑通
- [ ] 把 mini-react 仓库 push 到 GitHub
- [ ] 博客《React Fiber 是什么》发出来

## 常见错误

| 现象 | 原因 | 解法 |
| --- | --- | --- |
| `useState` 第二次返回还是初始值 | 没把 `oldHook` 从 `alternate` 取出来 | 检查 `wipFiber.alternate?.hooks?.[i]` |
| 输入框输入字符就丢焦点 | 输入触发 rerender，整个 input 被重建 | diff 时同 type 必须复用 dom，不要新建 |
| effect 永远不跑 cleanup | 没把上次的 `destroy` 通过 `oldHook` 传过来 | hook 上记 `destroy` 字段 |
| 列表删一项跑飞 | 没处理 DELETION 或没用 key | 走 `deletions` 队列 + key map |
| 函数组件挂到 DOM 报错 | `commitWork` 找父节点没跳过没 dom 的 fiber | `while(!p.dom) p = p.parent` |

## 资源
- big-react 课程：https://github.com/BetaSu/big-react （强烈推荐看完）
- 卡颂《React 设计原理》：电子书或纸质
- A Cartoon Intro to Fiber：https://www.youtube.com/watch?v=ZCuYPiUIONs
- React 源码导读：https://react.iamkasong.com/
- React Lane 模型源码：`packages/react-reconciler/src/ReactFiberLane.js`

---

## 第 04 章 完结语

到这里，第 04 章 React 全部结束。你应当已经做到：
1. 能用 React + TS + 路由 + 状态库 写出生产级 SPA。
2. 能用 Next.js 写 SSR/SSG/RSC + Server Actions 全栈应用。
3. 能写测试（70%+ 覆盖）+ 跑 CI。
4. 能用 Profiler 定位性能问题并优化。
5. 能手写 ~500 行 mini-react 跑通真实小应用。

下一章见——**第 05 章 Vue 生态**。
