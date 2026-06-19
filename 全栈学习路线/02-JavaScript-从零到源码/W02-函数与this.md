---
title: W02 函数与 this
chapter: 02
week: 2
duration: 7 天 / 每天 2-3 小时
prerequisite: W01 已掌握；能在 Node 跑代码
tags: [JavaScript, 函数, this, 箭头函数, 高阶函数]
---

# W02 - 函数与 this

> 本周目标：把 JS 函数玩明白。重点是 `this` 的 5 种绑定（这是 JS 区别于 Python 的最大坑），以及箭头函数为何不绑 this。结束时你应该能手写 `myCall / myApply / myBind`。

## 1. 函数声明 vs 函数表达式

```javascript
// 声明式：会被"提升"到顶部
function add(a, b) { return a + b; }
console.log(add(1, 2)); // 即使写在前面也能调用

// 表达式：不提升
const sub = function(a, b) { return a - b; };
// console.log(sub(1, 2)); 写在 const 前会报 ReferenceError
```

| 形式 | 提升 | 推荐场景 |
|------|------|---------|
| 声明 `function f(){}` | 整个提升 | 顶层工具函数 |
| 表达式 `const f = function(){}` | 不提升 | 大多数情况 |
| 箭头 `const f = () => {}` | 不提升 | 短小函数、回调 |

> **类比 Python**：Python 的 `def f():` 必须先定义后调用，类似 JS 的"表达式"。JS 的"声明式提升"是独有的小坑。

## 2. 箭头函数

```javascript
const add = (a, b) => a + b;            // 单表达式自动 return
const square = x => x * x;              // 单参数可省括号
const log = () => console.log('hi');    // 无参数必须写 ()
const make = x => ({ value: x });       // 返回对象字面量要套 ()
```

箭头函数 vs 普通函数 5 大区别：
1. **没有自己的 `this`**（继承外层）
2. **没有 `arguments`**（用 `...args` 替代）
3. **不能用作构造函数**（`new` 会报错）
4. **没有 `prototype`**
5. **不能 `yield`**（不能写 generator）

## 3. 默认参数 + 剩余参数

```javascript
function greet(name = '陌生人', greeting = '你好') {
  return `${greeting}, ${name}!`;
}
greet();                  // '你好, 陌生人!'
greet('Tom');             // '你好, Tom!'

// 剩余参数：把"剩下的"打包成数组
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);          // 10
```

> 类比 Python：`def greet(name='陌生人')` 一样，`*args` ↔ `...nums`。

## 4. 解构赋值

```javascript
// 数组解构
const [a, b, c] = [1, 2, 3];
const [first, ...rest] = [1, 2, 3, 4];   // first=1, rest=[2,3,4]
const [x = 10] = [];                      // 默认值 x=10

// 对象解构
const { name, age } = { name: 'Tom', age: 18 };
const { name: n2 } = { name: 'Tom' };     // 重命名 n2='Tom'
const { city = '未知' } = {};              // 默认值

// 函数参数直接解构
function showUser({ name, age = 0 }) {
  console.log(name, age);
}
showUser({ name: 'Tom' });    // Tom 0
```

> 类比 Python：`a, b, c = [1,2,3]` 类似数组解构。对象解构 Python 没有，是 JS 的便利之一。

## 5. 高阶函数三剑客

"高阶函数"= 接受函数为参数，或返回函数。

```javascript
const nums = [1, 2, 3, 4, 5];

// map：一对一变换
nums.map(x => x * 2);                   // [2,4,6,8,10]

// filter：筛选
nums.filter(x => x % 2 === 0);          // [2,4]

// reduce：累积成单值
nums.reduce((acc, x) => acc + x, 0);    // 15

// 链式
const result = nums
  .filter(x => x > 2)        // [3,4,5]
  .map(x => x * 10)          // [30,40,50]
  .reduce((a, b) => a + b);  // 120
```

> 类比 Python：`map/filter/reduce` 概念一致，但 Python 更推荐列表推导 `[x*2 for x in nums]`。JS 没有列表推导，链式 API 是首选。

## 6. `this` 的 5 种绑定（JS 最大坑）

> Python 的 `self` 必须显式写在第一个参数；JS 的 `this` 是隐式的，调用方式决定 `this` 指向。

### 规则 1：默认绑定（普通调用）
```javascript
function show() { console.log(this); }
show();
// 严格模式 → undefined
// 非严格 → window / global
```

### 规则 2：隐式绑定（作为方法调用）
```javascript
const user = {
  name: 'Tom',
  hi() { console.log(this.name); }
};
user.hi();        // 'Tom'   this = user
const fn = user.hi;
fn();             // undefined  this 丢了！
```

### 规则 3：显式绑定（call / apply / bind）
```javascript
function intro(city, hobby) {
  console.log(`${this.name} 来自 ${city}，喜欢 ${hobby}`);
}
const tom = { name: 'Tom' };

intro.call(tom, '北京', '写代码');         // 参数挨个传
intro.apply(tom, ['北京', '写代码']);      // 参数装数组
const bound = intro.bind(tom, '北京');     // 返回新函数，this 锁死
bound('写代码');                            // 后续可继续传参
```

| 方法 | 立即调用 | 参数形式 | 返回 |
|------|---------|---------|------|
| `call` | ✅ | 逐个 | 调用结果 |
| `apply` | ✅ | 数组 | 调用结果 |
| `bind` | ❌ | 逐个 | 新函数 |

### 规则 4：new 绑定
```javascript
function User(name) { this.name = name; }
const u = new User('Tom');
// new 自动：1) 创建空对象  2) this 指向它  3) 执行函数体  4) 返回它
console.log(u.name); // 'Tom'
```

### 规则 5：箭头函数（继承外层）
```javascript
const obj = {
  name: 'Tom',
  fn1: function() { return () => this.name; },  // 箭头继承 fn1 的 this
  fn2: () => this.name,                          // 箭头继承"定义时"外层
};
obj.fn1()();    // 'Tom'  内层箭头继承外层 fn1 的 this（=obj）
obj.fn2();      // undefined  fn2 的外层是全局，没有 name
```

### 优先级：new > 显式 > 隐式 > 默认

## 7. 手写 `myBind`（面试高频）

```javascript
Function.prototype.myBind = function(ctx, ...preArgs) {
  const fn = this;                       // this 是要 bind 的原函数
  return function(...laterArgs) {
    return fn.apply(ctx, [...preArgs, ...laterArgs]);
  };
};

function intro(city, hobby) {
  return `${this.name}@${city}-${hobby}`;
}
const bound = intro.myBind({ name: 'Tom' }, '北京');
bound('写代码');   // 'Tom@北京-写代码'
```

`myCall / myApply` 类似（思路：把函数挂到 ctx 上调用，再删掉）：

```javascript
Function.prototype.myCall = function(ctx, ...args) {
  ctx = ctx ?? globalThis;
  const key = Symbol();
  ctx[key] = this;
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};
```

## 8. 立即执行函数 IIFE

```javascript
(function() {
  console.log('我立即执行');
})();

(() => console.log('箭头版 IIFE'))();
```

历史用途：模块化（在没有模块系统的时代，靠 IIFE 隔离作用域）。今天用 ES Module 就行。

## 9. 实战练习

### 练习 1：手写 EventEmitter（Node 同款）
```javascript
class EventEmitter {
  constructor() { this.events = {}; }
  on(name, fn) {
    (this.events[name] ??= []).push(fn);
  }
  emit(name, ...args) {
    (this.events[name] || []).forEach(fn => fn(...args));
  }
  off(name, fn) {
    if (!this.events[name]) return;
    this.events[name] = this.events[name].filter(f => f !== fn);
  }
}
const bus = new EventEmitter();
bus.on('msg', x => console.log('收到', x));
bus.emit('msg', 'hello');   // 收到 hello
```

### 练习 2：this 谜题
```javascript
const obj = {
  x: 10,
  fn: function() { return this.x; },
  arrow: () => this.x,
};
console.log(obj.fn());                    // 10
console.log(obj.arrow());                 // undefined（this 指全局）
const f = obj.fn;
console.log(f());                          // undefined（默认绑定）
console.log(obj.fn.call({ x: 99 }));       // 99
```

### 练习 3：链式调用计算器
```javascript
class Calc {
  constructor(v = 0) { this.v = v; }
  add(x) { this.v += x; return this; }
  sub(x) { this.v -= x; return this; }
  mul(x) { this.v *= x; return this; }
  result() { return this.v; }
}
new Calc(10).add(5).sub(3).mul(2).result(); // 24
```

## 周末 Checklist

- [ ] 能默写箭头函数 5 大特性
- [ ] 看到任意函数能立即说出 `this` 指向
- [ ] 手写过 `myBind` 至少一次（不看答案）
- [ ] 能用 `map/filter/reduce` 链式处理数据
- [ ] 理解解构 + 默认参数 + 剩余参数

## 常见错误

1. **回调里 this 丢失**：`btn.onclick = obj.handle` → 用 `obj.handle.bind(obj)` 或箭头包裹
2. **箭头函数当方法**：`obj = { fn: () => this.x }` 永远拿不到 obj
3. **箭头函数当构造器**：`new (() => {})()` 直接报错
4. **`forEach` 不能 break**：用 `for...of` 或 `some/every`

## 资源

- 你不知道的 JavaScript（上卷）第 2 部分 this
- MDN this：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this
