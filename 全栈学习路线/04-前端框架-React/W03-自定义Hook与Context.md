---
chapter: 04
week: 3
duration: 7d
tags: [react, custom-hook, context, theme, i18n, auth, dark-mode]
---

# W03 自定义 Hook 与 Context

## 0. 什么时候该写自定义 Hook

判断公式：**两个组件里有 ≥3 行 useEffect / useState 写得几乎一样** → 抽成 `useXxx`。

类比：自定义 Hook = 「**逻辑乐高块**」。组件不再为"怎么订阅 / 怎么发请求 / 怎么读 storage"操心，调用一句 `useFetch(url)` 即可。

**两条命名约定**：
1. 必须以 `use` 开头（让 ESLint 能识别 Hook 规则）
2. 返回值要么是数组（位置无歧义时，模仿 useState），要么是对象（多个值时，避免顺序错）

```tsx
// 推荐
const [data, loading] = useFetch(url)        // 简单
const { data, loading, error, refetch } = useFetch(url)  // 字段多
```

---

## 1. useLocalStorage

把 `useState` 持久化到 localStorage。

```tsx
// src/hooks/useLocalStorage.ts
import { useState, useEffect } from 'react'

export function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const raw = localStorage.getItem(key)
    if (raw === null) return initial
    try { return JSON.parse(raw) as T } catch { return initial }
  })

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])

  return [value, setValue] as const
}
```

使用：

```tsx
const [theme, setTheme] = useLocalStorage<'light'|'dark'>('theme', 'light')
```

刷新后 theme 还在。

---

## 2. useDebounce

```tsx
// src/hooks/useDebounce.ts
import { useState, useEffect } from 'react'

export function useDebounce<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value)
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay)
    return () => clearTimeout(id)
  }, [value, delay])
  return debounced
}
```

```tsx
const [q, setQ] = useState('')
const dq = useDebounce(q, 400)
useEffect(() => { /* 用 dq 发请求 */ }, [dq])
```

---

## 3. useFetch（loading / error / abort）

```tsx
// src/hooks/useFetch.ts
import { useState, useEffect } from 'react'

interface State<T> { data: T | null; loading: boolean; error: Error | null }

export function useFetch<T>(url: string) {
  const [state, setState] = useState<State<T>>({
    data: null, loading: true, error: null,
  })

  useEffect(() => {
    const ctrl = new AbortController()
    setState({ data: null, loading: true, error: null })

    fetch(url, { signal: ctrl.signal })
      .then(r => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`)
        return r.json() as Promise<T>
      })
      .then(data => setState({ data, loading: false, error: null }))
      .catch(error => {
        if (error.name === 'AbortError') return
        setState({ data: null, loading: false, error })
      })

    return () => ctrl.abort()
  }, [url])

  return state
}
```

> 真实项目优先用 W04 的 **TanStack Query**，缓存、重试、并发都帮你处理了。这个 hook 是"懂原理"用的。

---

## 4. useEventListener

绑监听器、自动解绑：

```tsx
// src/hooks/useEventListener.ts
import { useEffect, useRef } from 'react'

export function useEventListener<K extends keyof WindowEventMap>(
  event: K,
  handler: (e: WindowEventMap[K]) => void,
  target: Window | HTMLElement = window,
) {
  const saved = useRef(handler)
  useEffect(() => { saved.current = handler }, [handler])

  useEffect(() => {
    const listener = (e: any) => saved.current(e)
    target.addEventListener(event, listener)
    return () => target.removeEventListener(event, listener)
  }, [event, target])
}
```

用 `useRef` 存 handler 是为了「**handler 变化不触发解绑**」。

```tsx
useEventListener('keydown', e => { if (e.key === 'Escape') onClose() })
```

---

## 5. useMediaQuery

响应式断点：

```tsx
// src/hooks/useMediaQuery.ts
import { useState, useEffect } from 'react'

export function useMediaQuery(query: string) {
  const [match, setMatch] = useState(() => window.matchMedia(query).matches)
  useEffect(() => {
    const mql = window.matchMedia(query)
    const onChange = () => setMatch(mql.matches)
    mql.addEventListener('change', onChange)
    return () => mql.removeEventListener('change', onChange)
  }, [query])
  return match
}
```

```tsx
const isMobile = useMediaQuery('(max-width: 768px)')
```

### 🔧 动手 1：写 `useToggle`

要求：返回 `[value, toggle, setTrue, setFalse]`。

<details><summary>答案</summary>

```tsx
import { useState, useCallback } from 'react'
export function useToggle(initial = false) {
  const [v, setV] = useState(initial)
  const toggle = useCallback(() => setV(x => !x), [])
  const setTrue = useCallback(() => setV(true), [])
  const setFalse = useCallback(() => setV(false), [])
  return [v, toggle, setTrue, setFalse] as const
}
```
</details>

---

## 6. Context = 跨层传值（再讲一次）

```tsx
const Ctx = createContext<Value | null>(null)

function useCtx() {
  const v = useContext(Ctx)
  if (!v) throw new Error('必须在 Provider 内使用')
  return v
}
```

`useCtx` 自定义 hook 是经典模式——把"必须在 Provider 内"的检查塞进去，使用方不用每次写。

---

## 7. Context 性能陷阱

⚠️ **Provider 的 value 引用变 → 所有消费者都重渲染**。

错误：

```tsx
// 每次 Layout 重渲染都生成新对象
<ThemeCtx.Provider value={{ theme, setTheme }}>
```

正确：

```tsx
const value = useMemo(() => ({ theme, setTheme }), [theme])
return <ThemeCtx.Provider value={value}>...</ThemeCtx.Provider>
```

### 7.1 拆分 Context

读得多的"值"和读得少的"setter"分开：

```tsx
const ThemeValueCtx = createContext<'light'|'dark'>('light')
const ThemeSetterCtx = createContext<(t: 'light'|'dark') => void>(() => {})
```

只读组件订阅 Value，按钮组件订阅 Setter——切主题不会让按钮重渲。

### 7.2 use-context-selector

更精细：选 Context 中的某个字段，字段没变就不重渲：

```bash
pnpm add use-context-selector
```

```tsx
import { useContextSelector } from 'use-context-selector'
const theme = useContextSelector(Ctx, v => v.theme)   // 只订阅 theme
```

判断：**Context value 字段超过 3 个 + 字段更新频率不一样** → 用 selector 或 Zustand（W04）。

---

## 8. Theme Context 实战（暗黑模式）

```tsx
// src/context/ThemeContext.tsx
import { createContext, useContext, useMemo, useEffect } from 'react'
import { useLocalStorage } from '../hooks/useLocalStorage'

type Theme = 'light' | 'dark'
interface Ctx { theme: Theme; toggle: () => void }
const ThemeCtx = createContext<Ctx | null>(null)

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useLocalStorage<Theme>('theme', 'light')

  useEffect(() => {
    document.documentElement.dataset.theme = theme
  }, [theme])

  const value = useMemo<Ctx>(() => ({
    theme,
    toggle: () => setTheme(t => t === 'light' ? 'dark' : 'light'),
  }), [theme, setTheme])

  return <ThemeCtx.Provider value={value}>{children}</ThemeCtx.Provider>
}

export function useTheme() {
  const v = useContext(ThemeCtx)
  if (!v) throw new Error('useTheme must be inside ThemeProvider')
  return v
}
```

CSS：

```css
:root[data-theme='light'] { --bg: #fff; --fg: #111; }
:root[data-theme='dark']  { --bg: #111; --fg: #eee; }
body { background: var(--bg); color: var(--fg); }
```

按钮：

```tsx
function ThemeToggle() {
  const { theme, toggle } = useTheme()
  return <button onClick={toggle}>{theme === 'light' ? '🌙' : '☀️'}</button>
}
```

---

## 9. i18n Context 实战

```tsx
// src/context/I18nContext.tsx
import { createContext, useContext, useMemo } from 'react'
import { useLocalStorage } from '../hooks/useLocalStorage'

const dict = {
  zh: { hello: '你好', logout: '退出' },
  en: { hello: 'Hello', logout: 'Logout' },
}
type Lang = keyof typeof dict
type Key = keyof typeof dict['zh']

interface Ctx { lang: Lang; setLang: (l: Lang) => void; t: (k: Key) => string }
const I18nCtx = createContext<Ctx | null>(null)

export function I18nProvider({ children }: { children: React.ReactNode }) {
  const [lang, setLang] = useLocalStorage<Lang>('lang', 'zh')
  const value = useMemo<Ctx>(() => ({
    lang, setLang,
    t: (k) => dict[lang][k],
  }), [lang, setLang])
  return <I18nCtx.Provider value={value}>{children}</I18nCtx.Provider>
}

export const useI18n = () => {
  const v = useContext(I18nCtx); if (!v) throw new Error('I18nProvider missing')
  return v
}
```

正式项目用 **react-i18next** 或 **lingui**——这里只学原理。

---

## 10. Auth Context 实战

```tsx
// src/context/AuthContext.tsx
import { createContext, useContext, useMemo, useState } from 'react'

interface User { id: string; name: string }
interface Ctx {
  user: User | null
  login: (name: string, pwd: string) => Promise<void>
  logout: () => void
}
const AuthCtx = createContext<Ctx | null>(null)

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(() => {
    const raw = localStorage.getItem('user')
    return raw ? JSON.parse(raw) : null
  })

  const value = useMemo<Ctx>(() => ({
    user,
    login: async (name, pwd) => {
      // 真实项目走 API
      await new Promise(r => setTimeout(r, 500))
      const u = { id: '1', name }
      setUser(u)
      localStorage.setItem('user', JSON.stringify(u))
    },
    logout: () => {
      setUser(null)
      localStorage.removeItem('user')
    },
  }), [user])

  return <AuthCtx.Provider value={value}>{children}</AuthCtx.Provider>
}

export const useAuth = () => {
  const v = useContext(AuthCtx); if (!v) throw new Error('AuthProvider missing')
  return v
}
```

---

## 11. 综合实战：mini App

把三个 Context 套起来。

```tsx
// src/main.tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { ThemeProvider } from './context/ThemeContext'
import { I18nProvider } from './context/I18nContext'
import { AuthProvider } from './context/AuthContext'
import App from './App'
import './index.css'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <ThemeProvider>
      <I18nProvider>
        <AuthProvider>
          <App />
        </AuthProvider>
      </I18nProvider>
    </ThemeProvider>
  </StrictMode>,
)
```

```tsx
// src/App.tsx
import { useState } from 'react'
import { useTheme } from './context/ThemeContext'
import { useI18n } from './context/I18nContext'
import { useAuth } from './context/AuthContext'

export default function App() {
  const { theme, toggle } = useTheme()
  const { lang, setLang, t } = useI18n()
  const { user, login, logout } = useAuth()
  const [name, setName] = useState('')

  return (
    <div style={{ padding: 24 }}>
      <header style={{ display: 'flex', gap: 12 }}>
        <button onClick={toggle}>{theme === 'light' ? '🌙' : '☀️'}</button>
        <select value={lang} onChange={e => setLang(e.target.value as any)}>
          <option value="zh">中</option>
          <option value="en">EN</option>
        </select>
        {user
          ? <><span>{t('hello')}, {user.name}</span><button onClick={logout}>{t('logout')}</button></>
          : (
            <>
              <input value={name} onChange={e => setName(e.target.value)} placeholder="name" />
              <button onClick={() => login(name, '')}>Login</button>
            </>
          )}
      </header>
      <main><h1>{t('hello')}</h1></main>
    </div>
  )
}
```

🎉 一个能切换主题、切换语言、登录登出的 mini App 就完工了——刷新页面状态都还在。

### 🔧 动手 2

加一个 `<RequireAuth>{children}</RequireAuth>` 组件：未登录时显示 "请先登录"。

<details><summary>答案</summary>

```tsx
import { useAuth } from './context/AuthContext'
export function RequireAuth({ children }: { children: React.ReactNode }) {
  const { user } = useAuth()
  if (!user) return <p>请先登录</p>
  return <>{children}</>
}
```
</details>

---

## 周末检查

### ✅ Checklist
- [ ] 写过 ≥3 个自定义 Hook
- [ ] 知道 Context value 必须用 useMemo
- [ ] 会拆分 Context 优化性能
- [ ] 跑通 Theme + i18n + Auth 三件套
- [ ] 理解"什么逻辑该抽 Hook，什么不该"

### 🐛 错误清单
- "Cannot find Context" → 忘记用 Provider 包，或者 Provider 在子树外
- 切主题时整个页面闪烁重渲 → value 没 useMemo
- 自定义 Hook 不工作 → 没以 use 开头，或写在条件里
- 自定义 Hook 死循环 → 依赖数组里放了"每次新引用"的对象

### 📚 资源
- usehooks-ts：<https://usehooks-ts.com/> 现成 hook 仓库（学过原理后可以直接用）
- react-i18next：<https://react.i18next.com/>
- use-context-selector：<https://github.com/dai-shi/use-context-selector>
