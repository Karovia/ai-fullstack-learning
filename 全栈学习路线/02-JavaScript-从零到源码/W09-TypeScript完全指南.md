---
title: W09 - TypeScript 完全指南
chapter: 02-JavaScript
week: 9
difficulty: ⭐⭐⭐⭐⭐
estimated_hours: 18
tags: [TypeScript, 类型系统, 泛型, 类型体操]
prerequisites: [W08-模块系统与工程化]
created: 2026-06-19
---

# W09 - TypeScript 完全指南 🛡️

> 💡 **本周目标**：从"会写 JS"升级到"写有类型的 JS"。让 IDE 当你的副驾驶，编译期发现 90% 的 bug。

---

## ❓ 一、为什么需要 TypeScript？

> 📖 **类比**：JS 像没有质检的小作坊——东西做出来才知道哪里不对。TS 是给生产线加传感器，每一道工序都帮你检查。

```javascript
// JS：运行时才崩
function getName(user) {
  return user.name.toUpperCase();
}
getName(null);  // 💥 TypeError
```

```typescript
function getName(user: { name: string } | null) {
  return user?.name.toUpperCase();  // 强制处理 null
}
```

**真实案例**：Airbnb 引入 TS 后，38% 的生产事故被编译器提前发现。

---

## ⚙️ 二、安装 + tsconfig.json

```bash
pnpm add -D typescript tsx
pnpm tsc --init   # 生成 tsconfig.json
```

### 关键字段

```json
{
  "compilerOptions": {
    "target": "ES2022",          // 编译到哪个 ES 版本
    "module": "ESNext",          // 模块格式
    "moduleResolution": "Bundler",
    "strict": true,              // ⭐ 严格模式（必开）
    "noUncheckedIndexedAccess": true,  // arr[0] 自动加 undefined
    "esModuleInterop": true,     // CJS/ESM 互操作
    "skipLibCheck": true,        // 跳过 .d.ts 检查（提速）
    "paths": {                   // 路径别名
      "@/*": ["./src/*"]
    },
    "outDir": "./dist",
    "declaration": true          // 输出 .d.ts
  },
  "include": ["src"]
}
```

> ⚠️ `strict: true` 一上来就开！否则后期改一遍工程量惊人。

---

## 🧱 三、基础类型

```typescript
let s: string = '小明';
let n: number = 18;
let b: boolean = true;
let u: undefined = undefined;
let nl: null = null;
let v: void = undefined;       // 函数无返回
let nv: never = (() => { throw new Error(); })();  // 永远到不了
let any1: any = '什么都行';     // ❌ 等于关掉 TS
let unk: unknown = '更安全的 any'; // ✅ 必须收窄后才能用

// 数组
let arr1: number[] = [1, 2];
let arr2: Array<number> = [1, 2];

// 元组（定长 + 类型固定）
let tuple: [string, number] = ['张三', 18];

// 对象 / 字面量
let obj: { name: string; age?: number } = { name: '小明' };
```

### any vs unknown

```typescript
let a: any = 1;
a.foo.bar();      // ✅ 能编译，运行可能炸

let u: unknown = 1;
u.foo;            // ❌ 编译错
if (typeof u === 'string') u.toUpperCase();  // ✅ 收窄后可用
```

> 💡 **能用 unknown 就别用 any**。`unknown` 强制你做类型判断，安全得多。

---

## 🏷️ 四、枚举 enum

```typescript
enum Role { Admin, User, Guest }   // 0, 1, 2
Role.Admin;     // 0
Role[0];        // 'Admin'（反向映射）

// 字符串枚举（推荐）
enum Status { Pending = 'PENDING', Done = 'DONE' }

// const enum：编译时被替换成字面量，0 运行时开销
const enum Color { Red, Blue }
const c = Color.Red;   // 编译为 const c = 0;
```

> 💡 现代项目更喜欢用**字面量联合**：`type Status = 'pending' | 'done'`，更轻量。

---

## ➕ 五、联合 | 与交叉 &

```typescript
type ID = string | number;        // 联合：是其中之一
type Admin = User & { role: 'admin' };  // 交叉：两个都满足

function format(id: ID) {
  if (typeof id === 'string') return id.trim();
  return id.toFixed(2);
}
```

---

## 🔍 六、类型守卫（缩窄）

```typescript
// 1. typeof
if (typeof x === 'string') { /* x: string */ }

// 2. instanceof
if (err instanceof Error) { err.message; }

// 3. in
if ('age' in user) { user.age; }

// 4. 自定义守卫（最强大）
function isCat(a: Cat | Dog): a is Cat {
  return (a as Cat).meow !== undefined;
}
```

---

## 📐 七、interface vs type

| 维度 | interface | type |
|---|---|---|
| 对象/类形状 | ✅ | ✅ |
| 联合/交叉/字面量 | ❌ | ✅ |
| 重复声明合并 | ✅ | ❌ |
| 继承 | extends | & |
| 性能（编译） | 略快 | 略慢 |

```typescript
interface User { name: string }
interface User { age: number }   // 自动合并
// 等价于 { name: string; age: number }

type Status = 'a' | 'b';         // type 才能写联合
```

**经验法则**：

- **库的公开 API** → interface（用户可扩展）
- **内部工具类型 / 联合 / 工具组合** → type

---

## 🧬 八、泛型 Generics（核心！）

```typescript
function identity<T>(x: T): T { return x; }
identity<string>('hi');
identity(42);   // T 自动推导为 number

// 多参数
function pair<A, B>(a: A, b: B): [A, B] { return [a, b]; }

// 约束
function getKey<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 默认值
type ApiResp<T = unknown> = { code: number; data: T };
```

### 实战：类型安全 EventEmitter

```typescript
type Events = {
  login: { user: string };
  logout: void;
  error: Error;
};

class Emitter<E extends Record<string, any>> {
  private h: { [K in keyof E]?: ((p: E[K]) => void)[] } = {};
  on<K extends keyof E>(name: K, fn: (p: E[K]) => void) {
    (this.h[name] ??= []).push(fn);
  }
  emit<K extends keyof E>(name: K, p: E[K]) {
    this.h[name]?.forEach(fn => fn(p));
  }
}

const e = new Emitter<Events>();
e.on('login', p => p.user.toUpperCase());  // ✅
e.emit('login', { user: 'tom' });
e.emit('login', { user: 1 });              // ❌ 类型错
e.emit('xxx', null);                       // ❌ 不存在的事件
```

---

## 📞 九、函数重载

```typescript
function fn(x: string): string;
function fn(x: number): number;
function fn(x: any): any { return x; }

fn('hi');   // string
fn(42);     // number
```

---

## 🤯 十、条件类型 + infer

```typescript
type IsString<T> = T extends string ? true : false;
type A = IsString<'hi'>;   // true
type B = IsString<42>;     // false

// infer：在条件类型里"抓"一个类型出来
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type R = ReturnType<() => string>;   // string

// 提取数组元素
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type N = ArrayElement<number[]>;     // number

// 提取 Promise 值
type Awaited<T> = T extends Promise<infer V> ? Awaited<V> : T;
```

### 分布式条件类型

```typescript
type ToArray<T> = T extends any ? T[] : never;
type R = ToArray<string | number>;
// = string[] | number[]（联合类型逐个分发）
```

---

## 📝 十一、模板字面量类型

```typescript
type Greet = `Hello, ${string}!`;
const g: Greet = 'Hello, World!';

// 大小写转换
type Upper = Uppercase<'abc'>;       // 'ABC'

// 路由参数提取
type ExtractParam<T> = T extends `/:${infer P}/${infer R}`
  ? P | ExtractParam<`/${R}`>
  : T extends `/:${infer P}` ? P : never;

type Params = ExtractParam<'/:userId/post/:postId'>;
// 'userId' | 'postId'
```

---

## 🔧 十二、Mapped Types + 内置工具类型

```typescript
// 自己写一个 Partial
type MyPartial<T> = { [K in keyof T]?: T[K] };

// 自己写一个 Readonly
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
```

### 内置必背

| 工具 | 作用 |
|---|---|
| `Partial<T>` | 全部变可选 |
| `Required<T>` | 全部必填 |
| `Readonly<T>` | 全部只读 |
| `Pick<T, K>` | 挑出指定 key |
| `Omit<T, K>` | 排除指定 key |
| `Record<K, V>` | 字典 `{ [k: K]: V }` |
| `Exclude<T, U>` | 联合中去除 |
| `Extract<T, U>` | 联合中保留 |
| `NonNullable<T>` | 去掉 null/undefined |
| `ReturnType<T>` | 函数返回值类型 |
| `Parameters<T>` | 函数参数元组 |
| `Awaited<T>` | Promise 解包 |

```typescript
type User = { id: number; name: string; pwd: string };
type PublicUser = Omit<User, 'pwd'>;
type UserMap = Record<string, User>;
```

---

## 🎀 十三、装饰器（实验性 → Stage 3）

```typescript
function log(target: any, key: string, desc: PropertyDescriptor) {
  const orig = desc.value;
  desc.value = function (...args: any[]) {
    console.log(`call ${key}`, args);
    return orig.apply(this, args);
  };
}

class Api {
  @log
  get(id: number) { return { id }; }
}
```

> ⚠️ 旧装饰器需要 `experimentalDecorators: true`；TS 5+ 的新版规范不需要，但语法略有差异。

---

## 🏋️ 十四、综合实战

1. 完成 [type-challenges](https://github.com/type-challenges/type-challenges) easy 全部（13 题）+ medium 5 题
2. 把 W8 的 mini-lodash 全量类型化（10 个函数都要类型安全）
3. 写一个类型安全的 `pick(obj, ['a','b'])` —— 只能传 obj 里存在的 key

```typescript
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const out = {} as Pick<T, K>;
  keys.forEach(k => out[k] = obj[k]);
  return out;
}
const u = { id: 1, name: 'a', age: 18 };
pick(u, ['id', 'name']);   // ✅
pick(u, ['xxx']);          // ❌
```

---

## 📝 本周练习

1. 写 `DeepReadonly<T>` 让对象嵌套全部 readonly
2. 写 `DeepPartial<T>`
3. 写 `Get<O, P>`：`Get<{a:{b:1}}, 'a.b'>` = `1`
4. 用模板字面量实现一个 SQL 类型解析器（极简）

### 参考：DeepReadonly

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

---

## ✅ 自测 Checklist

- [ ] tsconfig 关键字段全懂
- [ ] 知道 any/unknown/never 区别
- [ ] 写得出泛型函数 + 约束
- [ ] 用 infer 抓返回值类型
- [ ] 解释分布式条件类型
- [ ] 至少做完 type-challenges easy

---

## 🐛 常见错误

| 现象 | 原因 |
|---|---|
| `Object is possibly 'null'` | 必须 `?.` 或类型守卫 |
| `xx is of type 'unknown'` | 收窄后再用 |
| 类型死循环 | 递归没有终止条件 |
| as 强转后运行时崩 | as 只骗编译器，不改值 |
| 泛型推导不对 | 加约束 `<T extends ...>` |

---

## 📚 资源

- [TS 中文手册](https://www.typescriptlang.org/zh/docs/)
- [Total TypeScript](https://www.totaltypescript.com/)（Matt Pocock）
- [type-challenges](https://github.com/type-challenges/type-challenges)
- 《Effective TypeScript》62 条建议

> 🎯 **下周预告**：W10 V8 引擎与性能优化——揭秘 JS 是怎么被"跑快"的。第 02 章压轴！
