---
title: W06 - DOM 与事件
chapter: 02-JavaScript
week: 6
difficulty: ⭐⭐⭐
estimated_hours: 14
tags: [JavaScript, DOM, BOM, 事件, 防抖, 节流, 事件委托]
prerequisites: [W05-异步与Promise]
created: 2026-06-19
---

# W06 - DOM 与事件 🌳

> 💡 **本周目标**：让 JavaScript 真正能"操控网页"——点按钮、改文字、加元素、监听用户行为。这是前端的"灵魂"。

---

## 🌍 一、DOM 是什么？

**DOM（Document Object Model）** = 把 HTML 这棵"树"翻译成 JS 能操作的对象。

> 📖 **类比**：HTML 像一栋房子的设计图，DOM 就是这栋房子的"3D 模型"。你用 JS 在模型上敲敲打打，浏览器把变化"重新渲染"成你看到的页面。

```html
<html>
  <body>
    <h1>标题</h1>
    <p class="msg">你好</p>
  </body>
</html>
```

对应的 DOM 树：

```
document
└── html
    └── body
        ├── h1 ("标题")
        └── p.msg ("你好")
```

### BOM = 浏览器对象模型

| 对象 | 作用 | 例子 |
|---|---|---|
| `window` | 顶层对象（全局） | `window.alert()` |
| `location` | 当前 URL | `location.href` |
| `navigator` | 浏览器信息 | `navigator.userAgent` |
| `history` | 历史记录 | `history.back()` |
| `screen` | 屏幕信息 | `screen.width` |

---

## 🔍 二、选择元素

```javascript
// 1. 单个：getElementById（最快）
const el = document.getElementById('app');

// 2. 单个 CSS 选择器
const btn = document.querySelector('.btn-primary');

// 3. 多个 CSS 选择器（返回 NodeList）
const items = document.querySelectorAll('.item');
items.forEach(it => console.log(it));

// 4. 老 API（很少用了）
document.getElementsByClassName('item');  // HTMLCollection（动态）
document.getElementsByTagName('div');
```

> ⚠️ **HTMLCollection vs NodeList**：前者是"动态"（DOM 变它也变），后者是"静态快照"。优先用 `querySelectorAll`。

---

## ✏️ 三、增删改查

### 创建 + 插入

```javascript
const div = document.createElement('div');
div.textContent = '我是新元素';
div.className = 'box';

document.body.appendChild(div);              // 追加到末尾
document.body.insertBefore(div, firstChild); // 插入到指定位置前
parent.replaceChild(newNode, oldNode);       // 替换
parent.removeChild(oldNode);                 // 删除

// 现代 API（更好用）
parent.append(div, '文本节点');   // 可一次插入多个 + 字符串
parent.prepend(div);             // 插到开头
oldNode.replaceWith(newNode);    // 替换自己
oldNode.remove();                // 删除自己
```

### 修改属性

```javascript
// classList（推荐）
el.classList.add('active');
el.classList.remove('hidden');
el.classList.toggle('open');
el.classList.contains('active');  // true/false

// dataset（自定义 data-* 属性）
// HTML: <div data-user-id="42" data-role="admin"></div>
el.dataset.userId;   // "42"  注意 camelCase
el.dataset.role;     // "admin"

// style（行内样式）
el.style.color = 'red';
el.style.backgroundColor = '#fff';

// 属性
el.setAttribute('aria-hidden', 'true');
el.getAttribute('href');
el.removeAttribute('disabled');
```

### 文本 vs HTML

| 属性 | 行为 | 安全性 |
|---|---|---|
| `textContent` | 纯文本（保留所有空白） | ✅ 安全 |
| `innerText` | 纯文本（受 CSS 影响，会触发重排） | ✅ 安全 |
| `innerHTML` | **解析 HTML** | ⚠️ XSS 风险 |

```javascript
el.textContent = '<b>你好</b>';   // 显示字面量 <b>你好</b>
el.innerHTML = '<b>你好</b>';     // 显示加粗的 你好

// 用户输入永远不要直接 innerHTML！
el.innerHTML = userInput;  // ❌ XSS！
el.textContent = userInput; // ✅
```

---

## 🎯 四、事件 = 用户与页面的对话

### 基本绑定

```javascript
const btn = document.querySelector('#btn');

function handleClick(e) {
  console.log('点了！', e);
}

btn.addEventListener('click', handleClick);
btn.removeEventListener('click', handleClick); // 注意：函数引用必须一致

// 第三个参数
btn.addEventListener('click', handler, {
  once: true,      // 只触发一次自动移除
  capture: true,   // 捕获阶段触发
  passive: true,   // 承诺不调 preventDefault（滚动优化）
});
```

### 事件对象 event

```javascript
btn.addEventListener('click', (e) => {
  e.target;         // 真正被点击的元素
  e.currentTarget;  // 绑定监听器的元素（= btn）
  e.type;           // "click"
  e.clientX, e.clientY;  // 鼠标坐标
  e.preventDefault();    // 阻止默认行为（如表单提交、链接跳转）
  e.stopPropagation();   // 阻止冒泡
});
```

> 📖 **target vs currentTarget**：你点 `<button><span>OK</span></button>` 的 span，`target` 是 span，`currentTarget` 是 button（事件实际绑在 button 上）。

---

## 🌊 五、事件流：捕获 → 目标 → 冒泡

```
        ┌──────────┐
   ⬇ 捕获     window
        │  document
        │  body
        │  div.outer
        │  button     ← 目标阶段
        │  div.outer
        │  body
        │  document
   ⬆ 冒泡     window
        └──────────┘
```

```javascript
outer.addEventListener('click', () => console.log('外层捕获'), true);  // 捕获
outer.addEventListener('click', () => console.log('外层冒泡'));        // 冒泡
inner.addEventListener('click', () => console.log('内层'));

// 点 inner 输出顺序：外层捕获 → 内层 → 外层冒泡
```

### 事件委托（必学）

> 💡 **核心思想**：把监听器绑在父元素上，通过 `e.target` 判断真正被点的子元素。

```javascript
// ❌ 笨办法：每个 li 都绑监听器（100 个就 100 个）
document.querySelectorAll('li').forEach(li => {
  li.addEventListener('click', () => { /* ... */ });
});

// ✅ 委托：父元素一个监听器搞定，新增 li 自动生效
ul.addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('点了', e.target.textContent);
  }
  // 更精准：matches
  if (e.target.matches('li.deletable')) {
    e.target.remove();
  }
});
```

**优点**：性能好、动态元素自动响应、内存占用低。

---

## 🖱️ 六、拖拽实战

```javascript
const box = document.querySelector('.draggable');
let offsetX, offsetY, dragging = false;

box.addEventListener('mousedown', (e) => {
  dragging = true;
  offsetX = e.clientX - box.offsetLeft;
  offsetY = e.clientY - box.offsetTop;
});

document.addEventListener('mousemove', (e) => {
  if (!dragging) return;
  box.style.left = (e.clientX - offsetX) + 'px';
  box.style.top = (e.clientY - offsetY) + 'px';
});

document.addEventListener('mouseup', () => dragging = false);
```

> ⚠️ `mousemove` 必须绑在 `document` 上（否则鼠标移出 box 就丢失）。

---

## ⏱️ 七、防抖 vs 节流（手写）

### 防抖（debounce）：只执行最后一次

> 📖 **场景**：搜索框输入完毕再请求、窗口 resize 完成才计算。

```javascript
function debounce(fn, delay = 300) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce((kw) => fetch(`/api?q=${kw}`), 500);
input.addEventListener('input', (e) => search(e.target.value));
```

### 节流（throttle）：固定频率执行

> 📖 **场景**：滚动加载更多、按钮防连点。

```javascript
function throttle(fn, delay = 300) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

| 对比 | 防抖 | 节流 |
|---|---|---|
| 触发 | 停止后才执行 | 固定间隔执行 |
| 场景 | 输入搜索、resize | 滚动、拖拽 |

---

## 🎬 八、动画利器 requestAnimationFrame

```javascript
function animate() {
  box.style.left = (parseFloat(box.style.left || 0) + 1) + 'px';
  if (parseFloat(box.style.left) < 500) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);
```

> 💡 **优势**：浏览器在下一帧绘制前执行（约 60fps），省电、流畅；标签页隐藏会自动暂停。

---

## 👀 九、三大 Observer

```javascript
// 1. 元素是否进入视口（图片懒加载、无限滚动）
const io = new IntersectionObserver((entries) => {
  entries.forEach(e => {
    if (e.isIntersecting) console.log('进入视口', e.target);
  });
});
io.observe(document.querySelector('.lazy'));

// 2. 元素尺寸变化
new ResizeObserver(entries => { /* ... */ }).observe(el);

// 3. DOM 变化（属性、子节点）
new MutationObserver(records => { /* ... */ })
  .observe(el, { childList: true, attributes: true, subtree: true });
```

---

## 🏋️ 十、综合实战：TodoList（事件委托版）

```html
<input id="input" placeholder="输入任务" />
<button id="add">添加</button>
<ul id="list"></ul>
```

```javascript
const list = document.querySelector('#list');
const input = document.querySelector('#input');

document.querySelector('#add').addEventListener('click', () => {
  if (!input.value.trim()) return;
  const li = document.createElement('li');
  li.innerHTML = `<span>${input.value}</span> <button class="del">删除</button>`;
  list.append(li);
  input.value = '';
});

// 一个监听器搞定所有 li 的删除（事件委托）
list.addEventListener('click', (e) => {
  if (e.target.matches('.del')) {
    e.target.parentElement.remove();
  }
});
```

---

## 📝 本周练习

1. 实现 `debounce` + `throttle`，并写出"前缘 + 后缘"两种触发模式。
2. 实现可拖拽 div（边界限制：不能拖出窗口）。
3. 用 `IntersectionObserver` 实现图片懒加载。
4. 长列表虚拟滚动：1 万条数据，只渲染可视区域 + buffer。
5. TodoList 升级：本地存储、编辑、过滤完成项。

### 参考答案（节流前缘+后缘）

```javascript
function throttle(fn, delay, { leading = true, trailing = true } = {}) {
  let last = 0, timer;
  return function (...args) {
    const now = Date.now();
    if (!last && !leading) last = now;
    const remain = delay - (now - last);
    if (remain <= 0) {
      clearTimeout(timer); timer = null;
      last = now;
      fn.apply(this, args);
    } else if (!timer && trailing) {
      timer = setTimeout(() => {
        last = leading ? Date.now() : 0;
        timer = null;
        fn.apply(this, args);
      }, remain);
    }
  };
}
```

---

## ✅ 自测 Checklist

- [ ] 能解释 `target` 与 `currentTarget` 的区别
- [ ] 知道事件流三阶段
- [ ] 会用事件委托优化大量子元素
- [ ] 手写防抖、节流
- [ ] 知道 `innerHTML` 的 XSS 风险
- [ ] 用过 IntersectionObserver

---

## 🐛 常见错误

| 现象 | 原因 |
|---|---|
| `removeEventListener` 移不掉 | 传入的函数引用不一致（箭头函数每次都新建） |
| 拖拽抖动 | 没在 document 上监听 mousemove |
| innerHTML 注入脚本 | 未转义用户输入 |
| 滚动卡顿 | 没用 passive: true 或没节流 |
| forEach NodeList 不行 | 老浏览器；用 `Array.from(nodeList)` |

---

## 📚 资源

- MDN [DOM 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)
- [JavaScript.info DOM 章节](https://zh.javascript.info/document)
- 张鑫旭《现代 Web 布局》

> 🎯 **下周预告**：W07 ES6+ 进阶——Symbol、Proxy、Generator、Map/Set、可选链……Vue 3 响应式的底层就在这里！
