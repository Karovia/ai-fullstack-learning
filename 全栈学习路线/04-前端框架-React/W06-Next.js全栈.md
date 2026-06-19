---
chapter: 04
week: 06
duration: 7天
tags: [react, nextjs, ssr, ssg, rsc, app-router, server-actions, vercel]
---

# W06 · Next.js 全栈

> 上周你已经能用 Vite + React + Router + Zustand 写一个 SPA。
> 这周升级：**一个 Next.js 项目同时干掉 前端 + 后端 + 部署**。

---

## 一、为什么需要 Next.js（30 分钟读懂）

纯 React (CRA / Vite) 的痛点：

| 痛点 | 表现 | Next.js 怎么解决 |
| --- | --- | --- |
| SEO 差 | 首屏空 HTML，爬虫抓不到 | SSR / SSG 直出 HTML |
| 首屏慢 | JS 下载完才渲染 | 服务端预渲染 |
| 路由要装库 | react-router 手动配 | 文件即路由 |
| 图片优化要自己做 | `<img>` 没懒加载 | `<Image>` 自动 webp + lazy |
| 字体闪烁 (FOUT) | 切字体时跳一下 | `next/font` 自托管 |
| 后端要另起 | 还得起 Express | API Routes / Server Actions |
| 部署麻烦 | 配 Nginx / Docker | `git push` → Vercel 上线 |

**类比**：纯 React 像"自己买零件装 PC"，Next.js 像"开箱即用的 Mac"。

### 三种渲染模式
- **CSR**（Client Side Rendering）：浏览器 JS 渲染。Vite 默认。
- **SSR**（Server Side Rendering）：每次请求服务端渲染 HTML。新闻、个性化页面。
- **SSG**（Static Site Generation）：构建时生成 HTML。博客、文档、营销页。
- **ISR**（Incremental Static Regeneration）：SSG + 定时再生成。
- **RSC**（React Server Components）：组件本身就在服务端跑，不打包到 bundle。Next 14+ 默认。

---

## 二、创建项目（5 分钟）

```bash
pnpm create next-app@latest my-blog
# 全部按推荐：TypeScript ✅ ESLint ✅ Tailwind ✅ App Router ✅ src/ ❌ alias @/* ✅
cd my-blog
pnpm dev   # http://localhost:3000
```

目录结构：

```
my-blog/
├─ app/                # ← App Router（核心）
│  ├─ layout.tsx       # 根布局（必有）
│  ├─ page.tsx         # 首页 /
│  └─ globals.css
├─ public/             # 静态资源（直接 / 访问）
├─ next.config.ts
└─ package.json
```

---

## 三、文件路由约定（背下来）

App Router 用**约定文件名**生成路由：

| 文件 | 作用 | 类比 |
| --- | --- | --- |
| `page.tsx` | 这个 URL 的页面 | 网页 |
| `layout.tsx` | 包裹子页面的布局 | 相框 |
| `loading.tsx` | 加载中骨架 | "加载中…" |
| `error.tsx` | 错误边界 | 备胎 |
| `not-found.tsx` | 404 | 备用门 |
| `template.tsx` | 类似 layout 但每次重新 mount | 一次性相框 |
| `route.ts` | API 路由（GET/POST 函数） | Express handler |

```
app/
├─ layout.tsx                # / 全局
├─ page.tsx                  # /
├─ blog/
│  ├─ layout.tsx             # /blog/* 共享侧边栏
│  ├─ page.tsx               # /blog
│  ├─ loading.tsx
│  └─ [slug]/                # 动态路由 /blog/abc
│     └─ page.tsx
├─ shop/
│  └─ [...slug]/             # 全捕获 /shop/a/b/c
│     └─ page.tsx
├─ (marketing)/              # Route Group：括号文件夹不进 URL
│  ├─ about/page.tsx         # /about
│  └─ pricing/page.tsx       # /pricing
└─ api/
   └─ hello/route.ts         # GET /api/hello
```

```tsx
// app/blog/[slug]/page.tsx
type Props = { params: Promise<{ slug: string }> }   // Next 15 起 params 是 Promise
export default async function Page({ params }: Props) {
  const { slug } = await params
  return <h1>文章：{slug}</h1>
}
```

---

## 四、Server Components vs Client Components（重点）

**默认所有组件都是 Server Component**（在服务端渲染，不发 JS 到浏览器）。
要用 `useState` / `useEffect` / 浏览器 API，必须在文件**第一行**写：

```tsx
"use client"
```

| 能力 | Server Component | Client Component |
| --- | --- | --- |
| `useState` / `useEffect` | ❌ | ✅ |
| 直接访问数据库 / fs / env | ✅ | ❌ |
| `async` 函数组件 | ✅ | ❌ |
| 监听 DOM 事件 | ❌ | ✅ |
| 体积进 bundle | ❌ | ✅ |

**类比**：Server Component = 厨房（直连仓库），Client Component = 餐厅（招呼客人）。
`"use client"` 是**边界**：边界以内的子组件全部变 client。

```tsx
// app/page.tsx —— Server Component
import Counter from './Counter'
import { db } from '@/lib/db'
export default async function Home() {
  const posts = await db.post.findMany()      // 直接查数据库！
  return (
    <main>
      <h1>共 {posts.length} 篇</h1>
      <Counter />                              {/* 把交互"岛屿"嵌进来 */}
    </main>
  )
}

// app/Counter.tsx —— Client Component
"use client"
import { useState } from 'react'
export default function Counter() {
  const [n, setN] = useState(0)
  return <button onClick={() => setN(n + 1)}>{n}</button>
}
```

---

## 五、数据获取（fetch 被增强了）

Next 给原生 `fetch` 加了**缓存语义**：

```tsx
// 1. SSG（默认 force-cache）：构建时取一次，永远复用
const res = await fetch('https://api.example.com/posts')

// 2. SSR：每次请求都重取
const res = await fetch(url, { cache: 'no-store' })

// 3. ISR：60 秒后重新生成
const res = await fetch(url, { next: { revalidate: 60 } })

// 4. 带 tag，便于按标签精确刷新
const res = await fetch(url, { next: { tags: ['posts'] } })
```

### `generateStaticParams` —— 给动态路由预生成静态页

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getAllPosts()
  return posts.map(p => ({ slug: p.slug }))   // 构建时生成所有 /blog/xxx
}
```

---

## 六、Metadata API（SEO 一行搞定）

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'
export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: { images: [post.cover] },
  }
}
```

---

## 七、Server Actions（最革命的特性）

**直接在组件里写一个标了 `"use server"` 的函数，前端可以当事件处理器调它**——Next 自动帮你做"前端 → API → 后端"那一段。

```tsx
// app/blog/[slug]/Comment.tsx
import { db } from '@/lib/db'
import { revalidatePath } from 'next/cache'

async function addComment(formData: FormData) {
  "use server"                                         // ← 关键
  const text = formData.get('text') as string
  await db.comment.create({ data: { text } })
  revalidatePath(`/blog/${formData.get('slug')}`)      // 失效缓存，页面立刻更新
}

export default function CommentForm({ slug }: { slug: string }) {
  return (
    <form action={addComment}>
      <input type="hidden" name="slug" value={slug} />
      <textarea name="text" required />
      <button>提交</button>
    </form>
  )
}
```

无需写 `/api/comment` 接口，无需 `fetch`，无需 `useState` 处理 loading——全部由框架处理。

---

## 八、流式渲染 + Suspense

慢的部分**先返回骨架，数据好了再"流"过来**：

```tsx
// app/page.tsx
import { Suspense } from 'react'
export default function Home() {
  return (
    <>
      <Header />
      <Suspense fallback={<Skeleton />}>
        <SlowPostList />            {/* 这里 await db.query() 也不会卡住 Header */}
      </Suspense>
    </>
  )
}
```

---

## 九、Parallel / Intercepting Routes（高级）

- **Parallel Routes**：`@modal/page.tsx` 同时渲染多个槽（仪表盘场景）。
- **Intercepting Routes**：`(.)photo/[id]` 在当前页拦住一个跳转，把它弹成 Modal（Instagram 看图体验）。

---

## 十、中间件鉴权 `middleware.ts`

放在项目根目录，每个请求先过它一遍：

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(req: NextRequest) {
  const token = req.cookies.get('token')?.value
  if (req.nextUrl.pathname.startsWith('/admin') && !token) {
    return NextResponse.redirect(new URL('/login', req.url))
  }
}

export const config = { matcher: ['/admin/:path*'] }
```

---

## 十一、部署到 Vercel（5 分钟）

1. 把代码 push 到 GitHub。
2. 打开 https://vercel.com → New Project → 选你的仓库。
3. 点 Deploy。完。
4. 之后每次 `git push`，自动构建 + 上线。Pull Request 还会得到独立预览域名。

---

## 十二、实战：Markdown 博客 + 评论

**目标**：
- `/` 列出所有文章（SSG）
- `/blog/[slug]` 渲染 Markdown（SSG + ISR 60s）
- 文章底部有评论框（Server Action 写入 SQLite）

**关键依赖**：
```bash
pnpm add gray-matter remark remark-html better-sqlite3
```

**目录**：
```
content/posts/hello.md   → frontmatter + markdown
lib/posts.ts             → 读 content/、解析 md
lib/db.ts                → better-sqlite3 单例
app/page.tsx             → 列表
app/blog/[slug]/page.tsx → 详情 + Comment 组件
```

`lib/posts.ts` 关键代码：

```ts
import fs from 'node:fs/promises'
import path from 'node:path'
import matter from 'gray-matter'
import { remark } from 'remark'
import html from 'remark-html'

const DIR = path.join(process.cwd(), 'content/posts')
export async function getAllPosts() {
  const files = await fs.readdir(DIR)
  return Promise.all(files.map(async f => {
    const raw = await fs.readFile(path.join(DIR, f), 'utf8')
    const { data, content } = matter(raw)
    return { slug: f.replace(/\.md$/, ''), ...data, content } as any
  }))
}
export async function renderMd(md: string) {
  return (await remark().use(html).process(md)).toString()
}
```

---

## 周末 Checklist

- [ ] 跑通 `pnpm create next-app` 并理解每个目录
- [ ] 能解释 RSC / Client Component / `"use client"` 边界
- [ ] 写一个动态路由 `[slug]` 并用 `generateStaticParams`
- [ ] 用 `fetch` 三种缓存策略做对比实验
- [ ] 写一个 Server Action 向 SQLite 写数据 + `revalidatePath`
- [ ] 用 `Suspense` 做流式渲染
- [ ] `middleware.ts` 做一个 `/admin` 鉴权
- [ ] 部署到 Vercel，把链接发给朋友看
- [ ] 实战：Markdown 博客 + 评论功能上线

## 常见错误

| 报错 | 原因 | 解法 |
| --- | --- | --- |
| `useState only works in client component` | 漏写 `"use client"` | 文件首行加 |
| `Functions cannot be passed directly to Client Components` | 把 server 函数当 prop 传给 client | 包成 Server Action 或挪到 client |
| `Hydration failed` | 服务端 HTML ≠ 客户端首渲染 | 检查 `Date.now()`、随机数、`window` |
| `params should be awaited` | Next 15 起 params 是 Promise | `const { slug } = await params` |
| `fetch 一直返回旧数据` | 默认 force-cache | 加 `cache: 'no-store'` 或 `revalidate` |

## 资源
- 官方教程：https://nextjs.org/learn （强烈建议从头打一遍）
- App Router 文档：https://nextjs.org/docs/app
- RSC 解释：https://github.com/reactwg/server-components/discussions/4
- Vercel 部署：https://vercel.com/docs
