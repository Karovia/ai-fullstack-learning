---
title: 第 02 章 - JavaScript 从零到源码
chapter: 02
duration: 10 周
prerequisite: [[01-Python-从零到源码/01-Python总纲]]
tags: [JavaScript, 前端, 源码级]
---

# 第 02 章 · JavaScript 从零到源码（10 周）

## 🎯 章节目标

- 不靠 jQuery/React 手写动态网页
- 彻底搞懂 this / 闭包 / 原型链 / 事件循环 / Promise 内部实现
- 掌握 ES6+ 全部新特性，会用 TypeScript
- 能读懂部分 V8 设计文档，理解 JIT 编译思路
- 手写 mini-lodash + Promise A+ 规范实现

## 📅 周计划

| 周 | 主题 | 难度 | 关键产出 |
|---|------|------|----------|
| W1 | 语法 + 类型系统（值类型 vs 引用类型） | 入门 | 80 道基础题 |
| W2 | 函数：声明 vs 表达式、this、call/apply/bind | 入门→进阶 | 手写 bind |
| W3 | 闭包 + 作用域 + 执行上下文 | 进阶 | 闭包面试 10 题 |
| W4 | 原型链 + 继承（6 种方式 + class） | 进阶 | 手写 new / instanceof |
| W5 | 事件循环 + 异步：callback / Promise / async-await | 进阶 | 手写 Promise A+ |
| W6 | DOM + BOM + 事件机制 | 入门→进阶 | 手写事件委托/拖拽 |
| W7 | ES6+ 全特性：解构/Symbol/Iterator/Generator/Proxy | 进阶 | mini-lodash |
| W8 | 模块系统：CJS / ESM / UMD + 工程化 | 进阶 | 手写 require |
| W9 | TypeScript 完全指南 | 进阶 | 类型体操 30 题 |
| W10 | V8 引擎导读 + 性能优化 | 源码级 | V8 笔记 |

## 📖 详细内容大纲

### W1 - 语法与类型系统
- 7 种基本类型 + 1 种对象类型
- 类型判断：typeof / instanceof / Object.prototype.toString
- 类型转换（隐式 vs 显式）+ 经典坑题（`[] == ![]`）
- 严格模式 'use strict'
- **必看**：[现代 JS 教程 - JavaScript 基础](https://zh.javascript.info/first-steps)

### W2 - 函数
- 函数声明 vs 函数表达式 vs 箭头函数
- 参数：默认值、剩余参数、解构
- this 的 5 种绑定规则
- call / apply / bind 区别 + 手写
- **必读**：YDKJS - this & Object Prototypes

### W3 - 闭包与作用域
- 词法作用域 vs 动态作用域
- 执行上下文 + 作用域链
- 闭包定义 + 经典面试题（for-loop var/let）
- 内存泄漏与闭包

### W4 - 原型与继承
- `__proto__` vs `prototype`
- new 操作符做了什么 - 手写实现
- instanceof 原理 - 手写实现
- ES5 六种继承方式
- ES6 class 实质（Babel 编译产物）
- **必读**：YDKJS - this & Object Prototypes

### W5 - 异步与事件循环
- 浏览器 vs Node 事件循环差异
- 宏任务 vs 微任务（必背）
- Promise 状态机 + 链式调用
- **手写 Promise（符合 A+ 规范，过 Promises/A+ 测试套件）**
- async/await 是 Generator + Promise 的语法糖
- **必看**：[JavaScript Visualized: Event Loop](https://www.youtube.com/watch?v=eiC58R16hb8)

### W6 - DOM/BOM/事件
- DOM 增删改查（querySelector / classList / dataset）
- 事件捕获/冒泡/委托
- 事件循环与渲染（requestAnimationFrame）
- 防抖 / 节流 - 手写
- 拖拽 / 滚动加载 - 手写

### W7 - ES6+ 全特性
- 解构、扩展、模板字符串
- Symbol（迭代器协议）
- Iterator + Generator + for...of
- Proxy + Reflect（Vue 3 响应式核心）
- WeakMap / WeakSet / WeakRef
- **必读**：[阮一峰 ES6 入门](https://es6.ruanyifeng.com/)

### W8 - 模块系统
- IIFE → AMD/CMD → CommonJS → ES Modules
- 手写 require（含循环依赖处理）
- import / export + tree-shaking 原理
- 包管理：npm / pnpm / 工作空间

### W9 - TypeScript
- 基本类型 + 联合 + 交叉
- 泛型（核心！）
- 条件类型 + infer + 分布式
- 内置工具类型（Partial/Required/Pick/Omit/ReturnType）
- 装饰器（实验性）+ 元数据
- **必做**：[type-challenges](https://github.com/type-challenges/type-challenges) easy 全部

### W10 - V8 与性能
- V8 整体架构（Parser → Ignition → TurboFan / Maglev）
- 隐藏类 (Hidden Class) + 内联缓存 (IC)
- 垃圾回收（标记清除 + 分代）
- 性能优化：避免 deopt、减少重排重绘
- **必读**：[v8.dev/blog](https://v8.dev/blog)

## 📝 考核任务

- [ ] **T1（W1-2）**：80 道基础题（用 [JavaScript-Questions](https://github.com/lydiahallie/javascript-questions)）
- [ ] **T2（W2）**：手写 `Function.prototype.bind`，通过 5 个测试用例
- [ ] **T3（W3）**：解决 [闭包面试 10 题](https://github.com/Advanced-Frontend/Daily-Interview-Question)
- [ ] **T4（W4）**：手写 `new` / `instanceof` / `Object.create`
- [ ] **T5（W5）**：**手写 Promise，过完整的 [Promises/A+ 测试套件](https://github.com/promises-aplus/promises-tests)**
- [ ] **T6（W6）**：手写防抖 + 节流 + 拖拽 + 长列表虚拟滚动
- [ ] **T7（W7）**：手写 `mini-lodash`（10 个常用函数：cloneDeep/debounce/throttle/curry/flatten/get/set/merge/pick/omit）
- [ ] **T8（W8）**：手写 `require`，能加载本地 CJS 模块（含循环依赖）
- [ ] **T9（W9）**：完成 type-challenges easy 全部 + medium 至少 10 题
- [ ] **T10（W10）**：写一篇博客《V8 是如何把 JS 跑快的》（≥ 3000 字，配图）

## 🚀 章节大项目：`mini-lodash` + 在线代码沙盒

### 子项目 A：mini-lodash（npm 包）

实现一个 30 函数的工具库，要求：
- TypeScript 全量类型 + 完整 JSDoc
- ESM + CJS 双格式输出（用 tsup）
- Vitest 测试覆盖率 ≥ 90%
- Tree-shakable（每个函数独立导出）
- 发布到 npm，能 `pnpm i @yourname/mini-lodash`
- 文档站点（用 VitePress）

### 子项目 B：在线代码沙盒

做一个简化版 [CodePen](https://codepen.io/)：
- 三栏布局：HTML / CSS / JS 编辑器 + 实时预览
- 用 [Monaco Editor](https://github.com/microsoft/monaco-editor)
- iframe 隔离运行
- 代码 URL 分享（gzip + base64 编码到 hash）
- localStorage 保存历史
- **不准用 React/Vue**，纯原生 JS + Web Components

### 验收标准
- [ ] mini-lodash npm install 后可正常使用
- [ ] 沙盒部署到 Vercel/Netlify，能用公网 URL 访问
- [ ] 沙盒打开 < 1s，输入即时预览
- [ ] 通过 Lighthouse 性能 ≥ 90 分

### 进阶挑战
- mini-lodash 接入 [size-limit](https://github.com/ai/size-limit) 限制 bundle 体积
- 沙盒支持 npm 包加载（用 esm.sh CDN）

## 📚 推荐资源

### 入门
- 📖 [现代 JavaScript 教程（中文）](https://zh.javascript.info/) - 体系最完整
- 📖 [MDN JS 中文](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- 📘 《JavaScript 高级程序设计 (4th)》"红宝书" - 中文圈最经典

### 进阶
- 📘 [You Don't Know JS Yet](https://github.com/getify/You-Dont-Know-JS) ~185k star
- 📖 [阮一峰 ES6 入门](https://es6.ruanyifeng.com/)
- 🛠️ [JavaScript-Questions](https://github.com/lydiahallie/javascript-questions)
- 🛠️ [type-challenges](https://github.com/type-challenges/type-challenges)

### 源码级
- 🔗 [V8 官方文档](https://v8.dev/) + [V8 源码](https://github.com/v8/v8)
- 📖 [TC39 提案](https://github.com/tc39/proposals)
- 📘 《JavaScript 引擎权威指南》

### 中文社区
- 📖 [冴羽的博客](https://github.com/mqyqingfeng/Blog) - JS 深度系列必读
- 🛠️ [30-seconds-of-code](https://github.com/Chalarangelo/30-seconds-of-code)

---

✅ 完成项目 → 进入 [[03-HTML-CSS-现代Web/03-HTML-CSS总纲]]
