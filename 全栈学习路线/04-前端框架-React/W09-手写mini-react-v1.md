---
chapter: 04
week: 09
duration: 7天
tags: [react, fiber, mini-react, jsx, reconciler, scheduler, requestIdleCallback]
---

# W09 · 手写 mini-react v1（Fiber 核心）

> "**用过 ≠ 懂**"。本周用 ~300 行代码复刻一个能跑的小 React，
> 直到你能在白板上画出 Fiber 树。

---

## 一、为什么要手写

- 面试常问"React 怎么不卡"——背答案没用，写过才能答得自然。
- 看官方源码（几十万行）会劝退；先写**乞丐版**，再去看源码就像看注释。
- 真正吃透三件事：**JSX、Fiber、调度**。

---

## 二、React 整体架构（必背）

```
JSX
 │  Babel/SWC 编译
 ▼
React.createElement(...)        ← Element Tree（虚拟 DOM 描述）
 │  Reconciler（协调器）         ← 把 Element 转成 Fiber 链表，标记差异
 ▼
Fiber Tree
 │  Scheduler（调度器）          ← 决定何时干活，能中断
 ▼
Renderer（渲染器：react-dom / react-native）
 │  commit
 ▼
真实 DOM
```

类比一次装修：
- **JSX** = 户型图（你画的）
- **Element** = 设计稿（机器看的）
- **Reconciler** = 设计师（拆步骤）
- **Scheduler** = 项目经理（排期，可暂停）
- **Renderer** = 工人（真砸墙）

---

## 三、第 1 步：JSX 转换 + 自己实现 createElement

JSX 不是 React 语法，是 Babel/SWC 把它编译成函数调用：

```tsx
const el = <div id="x">hi <b>!</b></div>
// 等价于
const el = React.createElement('div', { id: 'x' }, 'hi ', React.createElement('b', null, '!'))
```

**自己实现**：

```ts
// src/mini-react.ts
type Props = Record<string, any> & { children: any[] }
type Element = { type: string | Function, props: Props }

function createElement(type: any, props: any, ...children: any[]): Element {
  return {
    type,
    props: {
      ...props,
      children: children.flat().map(c =>
        typeof c === 'object' ? c : createTextElement(c)
      ),
    },
  }
}

function createTextElement(text: string | number | boolean) {
  return {
    type: 'TEXT_ELEMENT',
    props: { nodeValue: text, children: [] },
  }
}

export default { createElement }
```

让 Vite/TS 用我们的版本：

```jsonc
// tsconfig.json
{ "compilerOptions": {
  "jsx": "react",
  "jsxFactory": "MiniReact.createElement"
}}
```

```tsx
// main.tsx
/** @jsx MiniReact.createElement */
import MiniReact from './mini-react'
const el = <div id="x">hi <b>!</b></div>
console.log(el)
```

控制台会看到一棵嵌套对象，这就是 Element Tree。

---

## 四、第 2 步：Element → DOM（递归版，演示问题）

```ts
function render(el: Element, container: HTMLElement) {
  const dom: any = el.type === 'TEXT_ELEMENT'
    ? document.createTextNode('')
    : document.createElement(el.type as string)

  const isProperty = (k: string) => k !== 'children'
  Object.keys(el.props).filter(isProperty).forEach(name => {
    if (name.startsWith('on')) {
      dom.addEventListener(name.toLowerCase().slice(2), el.props[name])
    } else {
      dom[name] = el.props[name]
    }
  })

  el.props.children.forEach(c => render(c, dom))
  container.appendChild(dom)
}
```

跑通了，但有大问题：**递归一旦开始就停不下来**。组件树一深，主线程被堵几百毫秒，按钮都点不动。
这就是 **Fiber 出现的原因**。

---

## 五、第 3 步：Fiber 是什么

把递归的"调用栈"，**改成可遍历的链表**。每个 Fiber 节点长这样：

```ts
type Fiber = {
  type: string | Function
  props: Props
  dom: HTMLElement | Text | null
  parent: Fiber | null
  child: Fiber | null
  sibling: Fiber | null
  alternate: Fiber | null   // 指向"上一次的自己"（双缓冲）
  effectTag?: 'PLACEMENT' | 'UPDATE' | 'DELETION'
}
```

三个指针（**画图记**）：

```
     A
    /
   B───C───D       (B.sibling = C, C.sibling = D)
  /
 E
```
- `child`：第一个孩子
- `sibling`：右边的兄弟
- `return` / `parent`：父节点

遍历顺序（深度优先）：A → B → E → C → D。每访问一个节点 = **一个工作单元 (unit of work)**。

---

## 六、第 4 步：双缓冲（current vs workInProgress）

> 一边渲染，一边页面照常跑，怎么不打架？**两棵树**。

- `current` 树 = 屏幕上当前在用的。
- `wipRoot` 树 = 正在构建的下一版。
- 构建完整棵 wip 后，**一次性切换**（commit），用户看到无闪烁更新。

```
current ──┐         wipRoot ──┐
   App    │             App   │
   /\     │             /\    │
  H  M    │            H  M'  │  ← M 改了，标 UPDATE
```

每个 wip Fiber 的 `alternate` 指回 current 同位置的 Fiber，diff 时方便比较。

---

## 七、第 5 步：requestIdleCallback 调度

浏览器每帧大概 16.6ms。React 的承诺：**渲染可以中断，让浏览器先处理用户输入**。

```ts
let nextUnitOfWork: Fiber | null = null
let wipRoot: Fiber | null = null

function workLoop(deadline: IdleDeadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
    shouldYield = deadline.timeRemaining() < 1   // 这一帧没空了，暂停
  }
  if (!nextUnitOfWork && wipRoot) commitRoot()
  requestIdleCallback(workLoop)
}
requestIdleCallback(workLoop)
```

> 真实 React 不用 `requestIdleCallback`（兼容性差、调度时机不准），自己写了 Scheduler 包基于 `MessageChannel`。这里我们用 `requestIdleCallback` 演示概念。

---

## 八、performUnitOfWork：一次干一点活

```ts
function performUnitOfWork(fiber: Fiber): Fiber | null {
  // 1. 给当前 fiber 创建 DOM（先不挂载到页面）
  if (!fiber.dom) fiber.dom = createDom(fiber)

  // 2. 给 children 创建 fiber，串成链表
  reconcileChildren(fiber, fiber.props.children)

  // 3. 返回下一个工作单元：child > sibling > parent.sibling
  if (fiber.child) return fiber.child
  let next: Fiber | null = fiber
  while (next) {
    if (next.sibling) return next.sibling
    next = next.parent
  }
  return null
}

function createDom(fiber: Fiber) {
  const dom: any = fiber.type === 'TEXT_ELEMENT'
    ? document.createTextNode('')
    : document.createElement(fiber.type as string)
  updateDom(dom, { children: [] }, fiber.props)
  return dom
}

function reconcileChildren(wipFiber: Fiber, children: Element[]) {
  let prevSibling: Fiber | null = null
  children.forEach((child, i) => {
    const newFiber: Fiber = {
      type: child.type,
      props: child.props,
      dom: null,
      parent: wipFiber,
      child: null,
      sibling: null,
      alternate: null,
      effectTag: 'PLACEMENT',
    }
    if (i === 0) wipFiber.child = newFiber
    else prevSibling!.sibling = newFiber
    prevSibling = newFiber
  })
}
```

**关键点**：**不要在 performUnitOfWork 里挂载 DOM**——挂一半被中断，用户会看到残缺页面。

---

## 九、第 6 步：commit 阶段

构建完整棵 wip 后，**一次同步**地把 DOM 全挂上去：

```ts
function commitRoot() {
  commitWork(wipRoot!.child)
  wipRoot = null
}

function commitWork(fiber: Fiber | null) {
  if (!fiber) return
  let parentFiber = fiber.parent
  while (parentFiber && !parentFiber.dom) parentFiber = parentFiber.parent  // 跳过函数组件
  const parentDom = parentFiber!.dom!

  if (fiber.effectTag === 'PLACEMENT' && fiber.dom) {
    parentDom.appendChild(fiber.dom)
  }
  // UPDATE / DELETION 等 v2 再加

  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

---

## 十、对外 API：render

```ts
function render(el: Element, container: HTMLElement) {
  wipRoot = {
    type: 'HOST_ROOT' as any,
    props: { children: [el] },
    dom: container,
    parent: null, child: null, sibling: null, alternate: null,
  }
  nextUnitOfWork = wipRoot
}

export default { createElement, render }
```

跑 demo（计数器先用回调写死，下周再实现 hooks）：

```tsx
/** @jsx MiniReact.createElement */
import MiniReact from './mini-react'

const container = document.getElementById('app')!
let n = 0
function rerender() {
  const el = (
    <div>
      <h1>计数: {n}</h1>
      <button onClick={() => { n++; rerender() }}>+1</button>
    </div>
  )
  MiniReact.render(el, container)
}
rerender()
```

打开浏览器，点按钮——成了。**你写出了一个能跑的 React。**

---

## 十一、画图理解（务必动手画）

练习：在纸上画下面这棵树的 Fiber 结构（含 child/sibling/return 三个指针），并写出 `performUnitOfWork` 的访问顺序：

```tsx
<App>
  <Header>
    <Logo/>
    <Nav/>
  </Header>
  <Main>
    <Article/>
  </Main>
</App>
```

---

## 周末 Checklist

- [ ] 自己实现的 `createElement` 输出对的 Element Tree
- [ ] 跑通递归版 `render`
- [ ] 能口头解释为什么递归 render 会卡
- [ ] 能在白板上画 Fiber 三指针
- [ ] 能写出 `performUnitOfWork` 的"child / sibling / parent" 三档返回
- [ ] 能解释双缓冲（current vs wip）
- [ ] commit 阶段一次性挂载 DOM 跑通
- [ ] 计数器 demo 能跑

## 常见错误

| 现象 | 原因 | 解法 |
| --- | --- | --- |
| JSX 不识别 `MiniReact` | tsconfig `jsx`/`jsxFactory` 没改，或文件没加 `/** @jsx */` 注释 | 二选一加上 |
| 页面渲染一半 | 在 `performUnitOfWork` 里 appendChild | 移到 `commitRoot` |
| children 是字符串报错 | 没把字符串转 TEXT_ELEMENT | `createTextElement` 别忘 |
| 死循环 | sibling/parent 链断了 | 检查 `prevSibling.sibling = newFiber` |

## 资源
- Build your own React (didact)：https://pomb.us/build-your-own-react/
- big-react（卡颂手把手版）：https://github.com/BetaSu/big-react
- React 源码导读：https://react.iamkasong.com/
- React 团队 RFC：https://github.com/reactjs/rfcs
