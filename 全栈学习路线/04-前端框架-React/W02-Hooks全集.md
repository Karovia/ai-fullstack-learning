---
chapter: 04
week: 2
duration: 7d
tags: [react, hooks, useState, useEffect, useMemo, useRef, useReducer, useContext]
---

# W02 Hooks 全集

## 0. 为什么有 Hooks

2018 年前，React 状态只能用 class 组件：

```tsx
// 旧时代痛苦
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { n: 0 }
    this.inc = this.inc.bind(this)   // 又要 bind this
  }
  inc() { this.setState({ n: this.state.n + 1 }) }
  componentDidMount() { /* 副作用 */ }
  componentDidUpdate() { /* 同样的副作用又要写一遍 */ }
  componentWillUnmount() { /* 还要清理 */ }
  render() { return <button onClick={this.inc}>{this.state.n}</button> }
}
```

两大痛：
1. **this 地狱**——每个方法都要 bind
2. **逻辑无法复用**——同一份"订阅 - 取消订阅"要在三个生命周期里写

Hook 把状态和副作用变成「**函数**」，可以像乐高一样自由组合：

```tsx
function Counter() {
  const [n, setN] = useState(0)
  useEffect(() => { /* 副作用 */ return () => { /* 清理 */ } }, [])
  return <button onClick={() => setN(n + 1)}>{n}</button>
}
```

---

## 1. Hook 三铁律

1. **必须在组件函数顶层调用**——不能写在 if/for/嵌套函数里
2. **只能在函数组件或自定义 Hook 里调用**——不能在普通函数里
3. **每次渲染顺序必须一致**——React 靠调用顺序识别哪个是哪个 state

类比：Hook 是「**抽屉编号**」，React 按顺序 1、2、3 取。如果有时跳过 2，就拿错抽屉里的东西。

错误示例：

```tsx
function Bad({ show }: { show: boolean }) {
  if (show) {
    const [n, setN] = useState(0)   // ❌ 条件调用
  }
}
```

正确：

```tsx
function Good({ show }: { show: boolean }) {
  const [n, setN] = useState(0)   // ✅ 顶层
  if (!show) return null
  return <p>{n}</p>
}
```

ESLint 插件 `eslint-plugin-react-hooks` 会自动检查，**必装**。

---

## 2. useState

### 2.1 基础

```tsx
const [count, setCount] = useState(0)
const [user, setUser] = useState<User | null>(null)
```

`useState(initial)` 返回一对：当前值 + 修改函数。

### 2.2 函数式更新

```tsx
// ❌ 连点 3 次只 +1（闭包陷阱）
const inc = () => {
  setCount(count + 1)
  setCount(count + 1)
  setCount(count + 1)
}

// ✅ 连点 3 次 +3
const inc = () => {
  setCount(c => c + 1)
  setCount(c => c + 1)
  setCount(c => c + 1)
}
```

**只要新值依赖旧值，就用函数式更新**——这是铁律。

### 2.3 惰性初始化

```tsx
// ❌ 每次渲染都跑一遍 expensiveCalc
const [data, setData] = useState(expensiveCalc())

// ✅ 只在第一次渲染跑一次
const [data, setData] = useState(() => expensiveCalc())
```

### 2.4 对象/数组要"产生新引用"

```tsx
// ❌ 不会触发渲染！React 用 Object.is 比较，引用没变
user.name = 'Tom'
setUser(user)

// ✅
setUser({ ...user, name: 'Tom' })
setList([...list, newItem])
setList(list.filter(x => x.id !== id))
```

### 🔧 动手 1：写一个 Stopwatch

要求：开始 / 暂停 / 重置，显示 ss.ms。

<details><summary>答案</summary>

```tsx
import { useState, useEffect, useRef } from 'react'
export default function Stopwatch() {
  const [ms, setMs] = useState(0)
  const [running, setRunning] = useState(false)
  const startRef = useRef(0)

  useEffect(() => {
    if (!running) return
    startRef.current = Date.now() - ms
    const id = setInterval(() => setMs(Date.now() - startRef.current), 16)
    return () => clearInterval(id)
  }, [running])

  return (
    <div>
      <h2>{(ms / 1000).toFixed(2)}s</h2>
      <button onClick={() => setRunning(r => !r)}>{running ? '暂停' : '开始'}</button>
      <button onClick={() => { setMs(0); setRunning(false) }}>重置</button>
    </div>
  )
}
```
</details>

---

## 3. useEffect

「**渲染后**」执行的副作用——发请求、订阅、改 document.title 等。

```tsx
useEffect(() => {
  document.title = `${count} 条消息`
}, [count])
```

### 3.1 依赖数组三种写法

```tsx
useEffect(() => { ... })          // 每次渲染都跑（几乎不用）
useEffect(() => { ... }, [])      // 只在挂载时跑一次（StrictMode 会两次）
useEffect(() => { ... }, [a, b])  // a 或 b 变就跑
```

### 3.2 清理函数

```tsx
useEffect(() => {
  const id = setInterval(tick, 1000)
  return () => clearInterval(id)   // 卸载或下次执行前调用
}, [])
```

类比：`useEffect` 是「**布置任务**」，return 的函数是「**收拾善后**」。订阅了就要退订，setInterval 了就要 clear，监听了 window event 就要 remove。

### 3.3 闭包陷阱

```tsx
function Chat({ roomId }: { roomId: string }) {
  useEffect(() => {
    const conn = connect(roomId)
    return () => conn.disconnect()
  }, [roomId])   // ⚠️ 必须把 roomId 写进依赖
}
```

如果忘写依赖，effect 里的 `roomId` 会**永远是第一次的值**——这是新人 80% bug 来源。

ESLint 插件 `react-hooks/exhaustive-deps` 会自动报警。

### 3.4 fetch 数据放 useEffect 的取舍

```tsx
useEffect(() => {
  let cancelled = false
  fetch('/api/users').then(r => r.json()).then(d => {
    if (!cancelled) setUsers(d)
  })
  return () => { cancelled = true }
}, [])
```

**坑**：竞态条件——快速切换 props 时，旧请求晚到会覆盖新数据。要么用 `cancelled` 标志，要么用 AbortController，要么直接上 **TanStack Query**（W04 讲，强烈推荐）。

---

## 4. useRef

两大用途：

### 4.1 访问 DOM

```tsx
const inputRef = useRef<HTMLInputElement>(null)
useEffect(() => { inputRef.current?.focus() }, [])
return <input ref={inputRef} />
```

### 4.2 存可变值（不触发渲染）

```tsx
const renderCount = useRef(0)
useEffect(() => { renderCount.current += 1 })
```

类比：useState 是「**展柜里的展品**」（动了通知大家），useRef 是「**抽屉里的笔记**」（自己改自己看，不通知）。

⚠️ **不要在渲染期读 ref.current** 来决定 UI——它不会触发更新。

---

## 5. useMemo / useCallback

```tsx
// useMemo：缓存计算结果
const expensive = useMemo(() => slowFn(items), [items])

// useCallback：缓存函数引用
const handle = useCallback(() => doThing(id), [id])
```

### 5.1 什么时候用？

**不要默认全包**！包裹本身有成本（要存依赖、对比依赖）。规则：

| 场景 | 用吗 |
|---|---|
| 普通的 onClick 处理 | ❌ 不用 |
| 计算 100ms+ 的值 | ✅ useMemo |
| 传给被 memo 包裹的子组件的 prop | ✅ useCallback / useMemo |
| 当 useEffect 依赖项 | ✅ useCallback |

### 5.2 React Compiler（2026 趋势）

React 19 配套的 **React Compiler**（实验稳定中）会**自动**给所有组件加 memo。装上后，你**几乎不用再写 useMemo/useCallback**。

```bash
pnpm add babel-plugin-react-compiler
```

```ts
// vite.config.ts
plugins: [react({ babel: { plugins: ['babel-plugin-react-compiler'] } })]
```

口诀：**"先把代码写清楚，性能交给 Compiler"**。

---

## 6. useReducer

state 多、转换复杂时，把"怎么变"集中到一个 reducer 里：

```tsx
type Action =
  | { type: 'add'; text: string }
  | { type: 'toggle'; id: number }
  | { type: 'remove'; id: number }

function reducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'add': return [...state, { id: Date.now(), text: action.text, done: false }]
    case 'toggle': return state.map(t => t.id === action.id ? { ...t, done: !t.done } : t)
    case 'remove': return state.filter(t => t.id !== action.id)
  }
}

function App() {
  const [todos, dispatch] = useReducer(reducer, [])
  return <button onClick={() => dispatch({ type: 'add', text: 'Hi' })}>+</button>
}
```

类比：useState 是「**散装变量**」，useReducer 是「**状态机**」——所有变更走 dispatch，可追溯可测试。

判断：**state 字段超过 3 个 + 转换逻辑复杂时**用 useReducer。

---

## 7. useContext

跨层传值，避免 props 钻孔：

```tsx
const ThemeCtx = createContext<'light' | 'dark'>('light')

function App() {
  return <ThemeCtx.Provider value="dark"><Page/></ThemeCtx.Provider>
}

function Page() {
  return <Title/>
}

function Title() {
  const theme = useContext(ThemeCtx)
  return <h1 className={theme}>Hi</h1>
}
```

⚠️ **Provider 的 value 变 → 所有用 useContext 的子组件都重渲**。性能优化看 W03。

---

## 8. React 18/19 新 Hook

### 8.1 useId

生成稳定唯一 id（SSR 不冲突）：

```tsx
const id = useId()
return <><label htmlFor={id}>姓名</label><input id={id} /></>
```

### 8.2 useTransition

把更新标记为"非紧急"，让浏览器先处理紧急的（比如打字）：

```tsx
const [isPending, startTransition] = useTransition()
const handleChange = (e) => {
  setQuery(e.target.value)              // 紧急（保证输入流畅）
  startTransition(() => {
    setResults(slowSearch(e.target.value))  // 非紧急（搜索结果可以慢一拍）
  })
}
```

### 8.3 useDeferredValue

延迟一个值，让昂贵渲染滞后：

```tsx
const deferredQuery = useDeferredValue(query)
return <SlowList q={deferredQuery} />
```

### 8.4 use（React 19 新）

可以「**直接 await Promise**」，配合 Suspense：

```tsx
function User({ promise }: { promise: Promise<User> }) {
  const user = use(promise)   // 自动 throw promise → Suspense 接住
  return <div>{user.name}</div>
}

<Suspense fallback="loading…">
  <User promise={fetchUser()} />
</Suspense>
```

`use` 还可以读 Context，且**能写在 if 里**（破例！）。

---

## 9. 实战：搜索框（防抖 + transition）

```tsx
// src/components/Search.tsx
import { useState, useEffect, useTransition } from 'react'

const fakeSearch = (q: string) =>
  new Promise<string[]>(r =>
    setTimeout(() => r(['apple', 'orange', 'banana'].filter(x => x.includes(q))), 200)
  )

export default function Search() {
  const [q, setQ] = useState('')
  const [debounced, setDebounced] = useState('')
  const [list, setList] = useState<string[]>([])
  const [pending, startTransition] = useTransition()

  // 防抖
  useEffect(() => {
    const id = setTimeout(() => setDebounced(q), 300)
    return () => clearTimeout(id)
  }, [q])

  useEffect(() => {
    if (!debounced) { setList([]); return }
    let cancelled = false
    fakeSearch(debounced).then(r => {
      if (!cancelled) startTransition(() => setList(r))
    })
    return () => { cancelled = true }
  }, [debounced])

  return (
    <div>
      <input value={q} onChange={e => setQ(e.target.value)} placeholder="搜水果" />
      {pending && <span>搜索中…</span>}
      <ul>{list.map(x => <li key={x}>{x}</li>)}</ul>
    </div>
  )
}
```

### 🔧 动手 2：模态框

写一个 `<Modal open onClose>{children}</Modal>`，要求：
1. open=false 不渲染
2. 点击遮罩关闭
3. ESC 关闭
4. 打开时禁止 body 滚动

<details><summary>答案</summary>

```tsx
import { useEffect } from 'react'
import { createPortal } from 'react-dom'

interface Props { open: boolean; onClose: () => void; children: React.ReactNode }

export function Modal({ open, onClose, children }: Props) {
  useEffect(() => {
    if (!open) return
    const handler = (e: KeyboardEvent) => e.key === 'Escape' && onClose()
    document.addEventListener('keydown', handler)
    document.body.style.overflow = 'hidden'
    return () => {
      document.removeEventListener('keydown', handler)
      document.body.style.overflow = ''
    }
  }, [open, onClose])

  if (!open) return null
  return createPortal(
    <div onClick={onClose}
         style={{ position: 'fixed', inset: 0, background: 'rgba(0,0,0,.5)',
                  display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
      <div onClick={e => e.stopPropagation()}
           style={{ background: '#fff', padding: 20, borderRadius: 8 }}>
        {children}
      </div>
    </div>,
    document.body
  )
}
```
</details>

---

## 周末检查

### ✅ Checklist
- [ ] 会用 useState 函数式更新避坑
- [ ] 知道 useEffect 依赖数组规则
- [ ] 写过带清理函数的 useEffect
- [ ] 会用 useRef 取 DOM 和存值
- [ ] 知道 useMemo/useCallback **何时不该用**
- [ ] useReducer 写过 Todo
- [ ] useContext 跨层传过值
- [ ] 试过 useTransition / useDeferredValue
- [ ] 装了 React Compiler 体验自动 memo

### 🐛 错误清单
- "useState 不更新" → 直接改了对象/数组的属性，没产生新引用
- "Maximum update depth exceeded" → useEffect 里 setState 又触发 effect 又 set
- "react-hooks/exhaustive-deps 警告" → 别简单加 // eslint-disable，先想清楚是不是闭包问题
- "Cannot read properties of null (reading 'focus')" → ref 在第一次渲染时还是 null

### 📚 资源
- React 文档"You Might Not Need an Effect"：<https://react.dev/learn/you-might-not-need-an-effect>
- React 文档"Synchronizing with Effects"：<https://react.dev/learn/synchronizing-with-effects>
- Dan Abramov《useEffect 完全指南》（中文版可搜）
