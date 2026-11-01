# 移动端开发者 → Vue 3 + TypeScript 前端工程师转型学习指导文档（V2.0 完整版）

**技术版本基线：Vue 3.5+ / TypeScript 5.x / Vite 7.x / Pinia 3.x / Node.js 22 LTS**

---

## 一、转型总览（保留 V1 精要）

### 1.1 经验资产映射

| 移动端经验 | 前端对应物 | 迁移成本 |
|---|---|---|
| Activity / Fragment / ViewController | Vue 组件 | ★ 低 |
| 生命周期回调 | Vue 生命周期钩子 | ★ 低 |
| Compose `mutableStateOf` / SwiftUI `@State` | Vue `ref()` 响应式 | ★ 低（心智一致） |
| RecyclerView 优化 | 虚拟列表 | ★★ 中 |
| Gradle / CocoaPods | pnpm / Vite | ★★ 中 |
| Retrofit + OkHttp | Axios 封装 | ★ 低 |
| MVVM 架构 | Vue 天然 MVVM | ★ 低 |
| 多线程 / 协程 | JS 事件循环 + 异步 | ★★★ 高（核心难点） |
| 封闭沙盒环境 | 浏览器兼容、跨域、安全 | ★★★ 高 |

**结论**：最大短板是 **①现代 JS 深度 ②CSS 文档流思维 ③浏览器与网络环境**。本版文档对这三块投入了最重的讲解篇幅。

### 1.2 技术栈基线

| 技术 | 版本 | 备注 |
|---|---|---|
| Vue | 3.5+ | Composition API + `<script setup>` 为主线 |
| TypeScript | 5.x | 岗位标配 |
| 构建 | Vite 7.x | 不学 Webpack 配置（懂原理即可） |
| 状态管理 | Pinia 3.x | 不学 Vuex |
| 路由 | Vue Router 4.x | — |
| UI 库 | Element Plus / Ant Design Vue 4 / Vant 4 | 中后台 / H5 |
| 测试 | Vitest + Vue Test Utils | — |

### 1.3 时间规划

```
阶段 0：环境搭建                    —— 第 1 周
阶段 1：Web 三件套 + 现代 JS        —— 第 2~4 周   ★ 重点扩写
阶段 2：TypeScript                 —— 第 5~7 周   ★ 扩写
阶段 3：Vue 3 核心                 —— 第 8~12 周  ★ 重点扩写
阶段 4：工程化与生态               —— 第 13~16 周 ★ 扩写
阶段 5：原理/性能/浏览器网络进阶    —— 第 17~22 周 ★ 重点扩写
阶段 6：项目实战                   —— 贯穿全程
```

---

## 二、阶段 0：环境搭建（第 1 周）

1. **nvm（或 fnm）管理 Node.js 22 LTS**：不同项目可能锁定不同 Node 版本，版本管理器是标配（类比 JDK 多版本管理）。
2. **pnpm**：`npm install -g pnpm`。相比 npm，它用硬链接共享依赖，安装快、磁盘省，Monorepo 支持好。
3. **VS Code 插件**：Vue - Official（Vue3+TS 官方语言支持）、ESLint、Prettier、Error Lens。
4. **Chrome DevTools**：Elements（查 DOM/样式，类比 Layout Inspector）、Console、Network（抓接口，类比 Charles）、Sources（断点调试）、Performance（性能分析，类比 Profiler）、Application（看 localStorage/Cookie）。

```bash
pnpm create vue@latest my-first-app
# 选择：TypeScript ✔ Router ✔ Pinia ✔ ESLint ✔ Prettier ✔（Vitest 可选）
cd my-first-app && pnpm install && pnpm dev
```

**实操**：画出项目加载链路图——`index.html` → `src/main.ts`（`createApp(App).use(router).use(pinia).mount('#app')`）→ `App.vue` → `<router-view>` 渲染路由组件。

---

## 三、阶段 1：Web 基础 + 现代 JavaScript（第 2~4 周）★ 核心补强

### 3.1 HTML 要点（3 天）

**语义化标签**：`header/nav/main/article/section/footer`。作用：可读性、SEO、无障碍（屏幕阅读器）。**把页面当“文档结构”而非“控件树”看待**，这是 Web 思维的第一个转变。

**表单**：`input` 的 `type`（text/password/number/email/date）、`required/minlength/pattern` 原生校验、`label for` 关联（点击文字聚焦输入框）。

**移动端关键 meta**：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<!-- width=device-width：视口宽=设备宽（CSS 像素，不是物理像素）
     initial-scale：初始缩放
     viewport-fit=cover：配合 env(safe-area-inset-*) 适配刘海屏 -->
```

**关键概念——设备像素比（DPR）**：`物理像素 = CSS 像素 × devicePixelRatio`。你熟悉的 dp/sp 概念与 CSS px 的换算逻辑本质相同：1 CSS px 在不同 DPR 屏幕上对应不同物理像素。

**图片优化**：`srcset` 多分辨率自适应、`loading="lazy"` 原生懒加载、`<picture>` 艺术指导。

---

### 3.2 CSS 深度讲解（约 1.5 周）

#### 3.2.1 盒模型 —— 一切布局的原子单位

```css
/* 标准盒模型：width 只算 content */
.box1 { box-sizing: content-box; width: 100px; padding: 20px; }
/* 实际占宽 = 100 + 20*2 + border*2 = 140px */

/* IE/边框盒模型（现代开发全局使用） */
* { box-sizing: border-box; }   /* 全局重置：width 就是最终占宽 */
.box2 { box-sizing: border-box; width: 100px; padding: 20px; }  /* 占宽 100px */
```

**实践经验**：项目入口必写 `box-sizing: border-box` 重置（类比 Android 的 `match_parent` 不含 padding 的痛，Web 直接一刀切解决）。

#### 3.2.2 文档流 —— 移动端开发者最大的思维转变

**核心认知差异**：原生移动布局是“我指定你放在哪”（约束布局/帧布局）；CSS 默认是“文档流”——**块级元素独占一行从上到下堆叠，行内元素从左到右排列，放不下自动换行**。脱离文档流只有三种方式：`float`（历史产物）、`position: absolute/fixed`、`flex/grid` 容器的子项。

#### 3.2.3 Flexbox —— 90% 场景的布局方案（精讲）

```css
.container {
  display: flex;
  /* 容器 6 属性 */
  flex-direction: row;          /* 主轴方向：row | row-reverse | column */
  justify-content: space-between; /* ★ 主轴对齐：flex-start|center|flex-end|space-between|space-around|space-evenly */
  align-items: center;          /* ★ 交叉轴对齐：stretch(默认撑满)|center|flex-start|flex-end|baseline */
  flex-wrap: wrap;              /* 换行策略 */
  gap: 16px;                    /* 子项间距（替代 margin 塌陷的痛苦） */
  align-content: center;        /* 多行时行组的对齐 */
}

.item {
  /* 子项 3 个关键属性 */
  flex-grow: 1;    /* 有剩余空间时的放大比例，0=不放大 */
  flex-shrink: 1;  /* 空间不足时的缩小比例，0=不缩小 */
  flex-basis: 0%;  /* 分配空间前的基准尺寸 */
  /* 缩写：flex: 1  ===  1 1 0%   → 所有 flex:1 的子项【等分】剩余空间（最常用）
     缩写：flex: auto ===  1 1 auto → 按内容比例分配
     缩写：flex: none ===  0 0 auto → 固定尺寸不缩放          */
  align-self: flex-end; /* 单个项覆盖容器的 align-items */
}
```

**必懂对比——`flex: 1` vs `flex: auto`**：三个 `flex:1` 的项宽度严格相等；三个 `flex:auto` 的项按**内容宽度比例**分配。做“等分布局”用前者，做“内容自适应+均分剩余”用后者。

**经典布局三件套**（每天都会写）：

```css
/* 1. 水平垂直居中 */
.center { display: flex; justify-content: center; align-items: center; }

/* 2. 移动端三段式（头部固定 + 内容滚动 + 底部固定） */
.page { display: flex; flex-direction: column; height: 100vh; }
.page header { flex: none; }
.page main   { flex: 1; overflow-y: auto; }
.page footer { flex: none; }

/* 3. 左侧固定 + 右侧自适应 */
.row { display: flex; }
.row .side { flex: none; width: 200px; }
.row .main { flex: 1; min-width: 0; } /* min-width:0 防止内容撑破（flex 子项默认 min-width:auto 的坑） */
```

#### 3.2.4 Grid —— 二维布局

```css
/* 后台仪表盘：侧边栏 220px + 内容区自适应，两行布局 */
.dashboard {
  display: grid;
  grid-template-columns: 220px 1fr;      /* 定义列轨道 */
  grid-template-rows: 60px 1fr;          /* 定义行轨道 */
  grid-template-areas:
    "sidebar header"
    "sidebar main";                       /* 命名区域，可读性极佳 */
  height: 100vh;
}
.dashboard .sidebar { grid-area: sidebar; }
.dashboard .header  { grid-area: header; }
.dashboard .main    { grid-area: main; }

/* 响应式卡片网格：一句话替代所有媒体查询（本阶段必背） */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  /* auto-fill：尽可能多列；minmax(240px,1fr)：每列最小 240px，剩余均分
     → 视口够宽自动 4 列，变窄自动 3/2/1 列，零媒体查询 */
  gap: 16px;
}
```

**Flex vs Grid 选择**：内容决定尺寸（一行按钮、导航条）→ Flex；需要先划分结构（页面骨架、表单网格）→ Grid。

#### 3.2.5 定位

```css
.pos-static  { position: static; }   /* 默认，文档流 */
.pos-relative { position: relative; top: 10px; } /* 相对自身原位置偏移，仍占原空间 —— 常用作 absolute 的锚点 */
.pos-absolute { position: absolute; top: 0; right: 0; } /* 相对最近的非 static 祖先定位，脱离文档流 */
.pos-fixed  { position: fixed; bottom: 0; } /* 相对视口 —— 底部吸底栏 */
.pos-sticky { position: sticky; top: 0; }   /* 滚动到阈值前正常、超过后固定 —— 表头吸顶常用 */
```

#### 3.2.6 BFC（块级格式化上下文）—— 面试必考 + 实战高频

**是什么**：一块独立渲染区域，内部布局不影响外部。
**触发条件**：`overflow` 非 `visible`、`float` 非 none、`position: absolute/fixed`、`display: flex/grid/inline-block`、根元素 html。
**解决的三大经典问题**：

```css
/* 问题1：父子 margin 塌陷（子元素的 margin-top 传给父元素） */
.parent { overflow: hidden; }  /* 父元素触发 BFC，隔断 margin 传递 */

/* 问题2：高度塌陷（子元素全部浮动导致父元素高度为0） */
.clearfix { overflow: hidden; } /* 或现代方案直接不用 float，用 flex */

/* 问题3：两栏自适应（左浮动固定宽，右元素被环绕）*/
.main { overflow: hidden; } /* 右侧触发 BFC，不与浮动元素重叠 */
```

#### 3.2.7 层叠上下文与 z-index —— “弹窗被盖住”的元凶

**规则**：`z-index` 只在**同一层叠上下文内**比较。以下元素会创建新的层叠上下文：根元素、`position` 非 static 且 z-index 非 auto、`opacity < 1`、`transform/filter` 非 none、`flex/grid` 子项设置了 z-index。
**典型坑**：弹窗 `z-index: 999` 仍被父级 `transform: translateX(0)` 的元素盖住——因为父级创建了层叠上下文，弹窗的 z-index 只在这个“小世界”里有效。**解法**：`<Teleport to="body">`（Vue 3 专门为此而生，见阶段 3）。

#### 3.2.8 CSS 优先级与选择器

```css
p { }                    /* 元素选择器 权重 0-0-1 */
.list-item { }           /* 类选择器 权重 0-1-0 */
#header { }              /* ID 选择器 权重 1-0-0 */
.nav a:hover { }         /* 组合权重相加 */
/* 权重：内联 style (1-0-0-0) > id > class/属性/伪类 > 元素/伪元素
   !important 最高（慎用）
   scoped 样式本质是属性选择器 [data-v-xxx]，权重按 class 级计算 */
```

#### 3.2.9 过渡与动画

```css
.btn { transition: all .3s ease; }  /* 变化时的过渡（状态A→B的补间） */
.btn:hover { transform: scale(1.05); background: #409eff; }

/* transform/opacity 走 GPU 合成层，不触发重排，性能最好 —— 动画首选 */
@keyframes spin { from { transform: rotate(0); } to { transform: rotate(360deg); } }
.loading { animation: spin 1s linear infinite; }
```

#### 3.2.10 CSS 工程化

```scss
// CSS 变量：主题系统基石（运行时可改，等价于原生主题色）
:root { --primary: #409eff; --danger: #f56c6c; }
.button { background: var(--primary); }

// SCSS：嵌套 + 变量 + mixin（$ 编译期变量，:root 运行时变量）
$spacing: 8px;
@mixin ellipsis($line: 1) {
  display: -webkit-box;
  -webkit-line-clamp: $line;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.card { padding: $spacing * 2; .title { @include ellipsis; font-weight: 600; } }

// BEM 命名：.块__元素--修饰符，解决命名冲突
.card {}
.card__title {}
.card--disabled {}
```

#### 3.2.11 移动端响应式适配三种方案对比

| 方案 | 原理 | 适用 |
|---|---|---|
| `rem` + `postcss-pxtorem` | 按设计稿写 px，构建时转 rem；JS 动态设 `html { font-size }` | H5 整体等比缩放（主流） |
| `vw` | `1vw = 视口宽/100`，postcss 直接 px→vw | 同上，纯 CSS 无需 JS |
| 媒体查询 + 断点 | 修改布局结构 | 需要在不同屏“变布局”而非“等比缩放” |

```css
/* 安全区适配（全面屏底部横条） */
.footer { padding-bottom: calc(12px + env(safe-area-inset-bottom)); }
```

**实操任务**：① 不查资料手写 Flex 三段式布局；② 用 `repeat(auto-fill, minmax())` 实现响应式卡片；③ 还原一个移动端 App 设置页（分组列表+右箭头+开关）。

---

### 3.3 现代 JavaScript 精讲（约 1 周，最高优先级）

#### 3.3.1 let/const 与作用域

```javascript
// var：函数作用域 + 变量提升（声明被提到顶部，值为 undefined）
console.log(a); // undefined（不报错！）
var a = 1;

// let/const：块级作用域 + 暂时性死区（声明前访问直接报错）
console.log(b); // ReferenceError
let b = 1;

// 经典闭包陷阱：var 导致循环变量共享
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));  // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i));  // 0 1 2
// 原因：let 每轮循环创建新的块级作用域，闭包捕获的是各自的 i
```

**原则**：默认全部 `const`，需要重新赋值才用 `let`，**永远不用 var**。

#### 3.3.2 闭包 —— 面试第一高频，必须讲透

**定义**：函数与其词法作用域（定义时可访问的变量）的组合。**内层函数引用了外层函数的变量，外层函数执行结束后这些变量也不会被回收**，因为内层函数还持有引用。

```javascript
// 最小示例
function createCounter() {
  let count = 0                    // 外层函数的私有变量
  return function () {
    return ++count                 // 内层函数捕获 count → 形成闭包
  }
}
const counter = createCounter()
counter() // 1 —— createCounter 早已执行完，count 却活着
counter() // 2

// 应用1：模块模式 / 私有变量（TS 的 private 之前，JS 靠这个）
function createUser(name) {
  let age = 18                                  // 外部无法直接访问
  return {
    getAge: () => age,
    grow: () => ++age,
  }
}

// 应用2：函数工厂（预置参数）
const multiply = (x) => (y) => x * y
const double = multiply(2)
double(5) // 10 —— x=2 被 double 闭包持有

// 应用3：防抖节流的实现基础（见 3.3.10）
// 应用4：Vue Composable 能"记住"状态的底层机制 —— useCounter() 返回的函数闭包住了内部 ref
```

**闭包的坑**：① 循环 + var（见上）；② 意外持有大对象导致**内存泄漏**——事件监听回调闭包住大数据且未解绑时，数据永远无法回收（这也是 Vue `onUnmounted` 必须清理监听的原因）。

#### 3.3.3 this 指向 —— 四规则 + 箭头函数

```javascript
// 规则1 默认绑定：独立函数调用。非严格模式 → window；严格模式 → undefined
function f1() { console.log(this) }  // window / undefined

// 规则2 隐式绑定：谁调用指向谁（看调用时点号左边）
const obj = { name: 'A', say() { console.log(this.name) } }
obj.say() // 'A'
const fn = obj.say; fn() // undefined —— 赋值后调用者丢失（隐式绑定丢失，高频坑）

// 规则3 显式绑定：call / apply / bind
obj.say.call({ name: 'B' })  // 'B'

// 规则4 new 绑定：指向新创建的对象
function Person(name) { this.name = name }
new Person('C').name // 'C'

// 优先级：new > 显式 > 隐式 > 默认

// 箭头函数：根本没有自己的 this，用定义时外层的 this（词法 this，等价 Kotlin Lambda）
const timer = {
  seconds: 0,
  start() {
    setInterval(() => { this.seconds++ }, 1000) // ✔ 箭头函数取 start() 的 this（即 timer）
    setInterval(function () { this.seconds++ }, 1000) // ✘ this 是 window
  }
}
```

**给移动端开发者**：Vue 3 `<script setup>` 中顶层 `this` 不存在（被刻意禁用），你遇到的 this 问题主要在**读旧代码和写原生 JS 工具函数**时。

#### 3.3.4 原型链与 class

```javascript
// JS 的"继承"：对象通过 __proto__ 链条查找属性
function Animal(name) { this.name = name }
Animal.prototype.eat = function () { return `${this.name} eats` }

const cat = new Animal('猫')
cat.eat()                       // 自身没有 eat → 沿 __proto__ 找到 Animal.prototype.eat
cat.__proto__ === Animal.prototype              // true
Animal.prototype.__proto__ === Object.prototype // true —— 链条终点
Object.prototype.__proto__ === null             // true —— 查找到此为止

// class 是原型机制的语法糖（Kotlin/Java 开发者直接用这个心智）
class Animal {
  #secret = '私有字段'          // 真私有（# 语法）
  constructor(name) { this.name = name }
  eat() {}                      // 挂在 prototype 上
  static create(name) { return new Animal(name) } // 静态方法挂类上
}
class Cat extends Animal {
  constructor(name, color) {
    super(name)                 // 必须先调用 super（等价先初始化父类构造）
    this.color = color
  }
}
```

#### 3.3.5 ES 模块（ESM）—— Vite 的基石

```javascript
// utils.js
export const formatPrice = (n) => `¥${(n / 100).toFixed(2)}`
export default class Request { }   // 每个文件最多一个默认导出

// main.js
import Request, { formatPrice } from './utils.js'  // 默认导出不用花括号，名字随意
import * as utils from './utils.js'                // 命名空间导入
```

**关键认知**：ESM 是**静态结构**（导入导出关系编译期确定）——这是 Tree Shaking（打包时删除未引用代码）的前提，也是 Vite 开发期“按需编译”快的原因。CommonJS（`require`）是运行时动态加载，Node 时代产物，看懂即可。

#### 3.3.6 解构、展开与默认值 —— Vue 代码里无处不在

```javascript
// 对象解构 + 重命名 + 默认值
const { name, age: userAge = 18, address: { city } = {} } = user

// 函数参数解构（组件 props 处理的基础）
function render({ list = [], loading = false, onClick }) {}

// 展开：合并对象（后者覆盖前者）、拷贝数组（浅拷贝）
const merged = { ...defaults, ...options }
const copy = [...arr, newItem]

// 剩余参数收集
const [first, ...rest] = [1, 2, 3]  // first=1, rest=[2,3]
```

#### 3.3.7 数组高阶方法 —— 前端的“集合流操作”

```javascript
const products = [
  { id: 1, name: '键盘', price: 299, stock: 10 },
  { id: 2, name: '鼠标', price: 99, stock: 0 },
  { id: 3, name: '显示器', price: 1999, stock: 5 },
]

products.map(p => ({ ...p, price: p.price * 0.8 }))      // 映射：返回新数组（不改原数组）
products.filter(p => p.stock > 0)                         // 过滤
products.find(p => p.id === 2)                            // 找第一个匹配项（无则 undefined）
products.findIndex(p => p.id === 2)                       // 找下标（无则 -1）
products.some(p => p.price > 1000)                        // 至少一个满足 → true
products.every(p => p.stock >= 0)                         // 全部满足 → true
products.reduce((sum, p) => sum + p.price, 0)             // 累加器：2397（第二参数是初始值，必传！）
products.sort((a, b) => b.price - a.price)                // 数字排序（默认按字符串排，高频坑）
products.flatMap(p => [p.name, p.price])                  // map 后拍平一层
Object.entries(product) / Object.keys / Object.values     // 对象↔数组转换
```

**易错点**：`sort/splice/reverse/push` 是**原地修改**（会改变响应式数据且触发更新），`map/filter/concat/slice` 返回新数组。React/Vue 生态提倡不可变数据风格，优先用后者。

#### 3.3.8 Promise —— 异步编程核心（重点讲透）

**三态**：`pending → fulfilled` 或 `pending → rejected`，**状态一旦落定不可逆**。

```javascript
// 创建
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve('成功'), 1000)
})

// 消费：then 链式调用，前一个的返回值是后一个的入参（链条会"传递"）
p.then(data => { console.log(data); return data + '!' })   // 返回值进入下一个 then
 .then(data => console.log(data))                          // '成功!'
 .catch(err => console.error(err))                         // 捕获链上任何一环的错误
 .finally(() => console.log('无论成败'))                    // 清理逻辑

// 关键细节：
// 1. then 中 return 普通值 → 包装成 fulfilled Promise 传下去
// 2. then 中 return 另一个 Promise → 链条会等待它落定（这就是串行的原理）
// 3. then 中 throw → 变成 rejected，被最近的 catch 捕获
```

**并发四兄弟（面试必考）**：

```javascript
const ps = [fetchA(), fetchB(), fetchC()]  // 三个已启动的 Promise

Promise.all(ps)         // 全部成功才成功；任何一个失败立即整体失败
// 典型场景：页面多个独立请求，需等全部完成才渲染

Promise.allSettled(ps)  // 永不失败；等全部落定，返回 [{status:'fulfilled',value} | {status:'rejected',reason}]
// 典型场景：批量上传，需要知道每一个的成败

Promise.race(ps)        // 第一个落定的（无论成败）决定结果
// 典型场景：请求超时控制 race([fetch(), timeout(3000)])

Promise.any(ps)         // 第一个【成功】的；全失败才失败（AggregateError）
// 典型场景：多 CDN 竞速，谁先返回用谁
```

#### 3.3.9 async/await —— Promise 的语法糖

```javascript
// async 函数的返回值自动包装为 Promise
// await = "等这个 Promise 落定再继续"，且不阻塞线程（让出执行权，事件循环继续转）
async function loadPage() {
  try {
    const user = await fetchUser()          // 串行：等 user 完成再请求 orders
    const orders = await fetchOrders(user.id)
    return { user, orders }
  } catch (err) {
    console.error(err)                      // await 的 rejection 可被 try/catch 捕获
    throw err                               // 需要继续抛出让调用方感知
  }
}

// ★ 高频优化点：无依赖的请求必须并行，不要傻等
async function loadPageBad()  { const a = await fetchA(); const b = await fetchB() } // 串行 2s
async function loadPageGood() {
  const [a, b] = await Promise.all([fetchA(), fetchB()])  // 并行 1s —— 同时发，一起等
}
```

**心智映射**：`await` ≈ Kotlin 的 `suspend fun` 调用 ≈ Swift 的 `async/await`。区别在于 JS 单线程，“挂起”期间事件循环继续处理其他任务。

#### 3.3.10 事件循环 —— 前端面试第一原理题

**模型**：JS 单线程执行 → 遇到异步 API 把回调交给 Web API 线程（定时器/网络/IO）→ 回调就绪后进入任务队列 → **调用栈清空后，先清空全部微任务，再取一个宏任务，如此循环**。

```
一次循环 = [执行一个宏任务] → [清空整个微任务队列] → (必要时渲染) → 下一个宏任务...

微任务：Promise.then/catch/finally、queueMicrotask、MutationObserver  ← 优先级更高
宏任务：setTimeout、setInterval、I/O、UI 事件回调
```

**经典输出题（必须能秒答）**：

```javascript
console.log('1')                                   // 同步
setTimeout(() => console.log('2'), 0)              // 宏任务（0ms 也不是立刻，要等下一轮）
Promise.resolve().then(() => console.log('3'))     // 微任务
console.log('4')                                   // 同步
// 输出：1 → 4 → 3 → 2

// 进阶：await 的本质（await x 后面的代码 = .then 回调 = 微任务）
async function f() {
  console.log('a')
  await Promise.resolve()
  console.log('b')        // 微任务！
  await null
  console.log('c')        // 又一个微任务
}
f()
setTimeout(() => console.log('d'), 0)
// 输出：a → b → c → d ？
// 错！正确：a → b → c → d 中间的顺序是 a, b, c, d？
// 精确顺序：a（同步）→ [微任务清空: b] → [微任务: c] → d（宏任务）
// 即 a, b, c, d。原因：每个 await 都把后续代码注册为一个微任务。
```

**实战意义**：① `Vue.nextTick` 就是用微任务实现的；② 长任务会阻塞渲染导致掉帧，拆分方法就是把大计算切片丢进宏任务或用 `requestIdleCallback`。

#### 3.3.11 事件模型：冒泡、捕获、委托

```javascript
// 事件流：捕获（window→目标）→ 目标 → 冒泡（目标→window）
// addEventListener 第三参数：true=捕获阶段监听（默认 false 冒泡阶段）

// 事件委托：利用冒泡，把 N 个子元素的监听挂到父元素上（列表上千项时的性能关键）
ul.addEventListener('click', (e) => {
  const li = e.target.closest('li')     // 找到实际被点的后代
  if (!li) return
  console.log('点击了', li.dataset.id)
})

// 阻止冒泡：e.stopPropagation()   阻止默认行为：e.preventDefault()
// Vue 中的对应：@click.stop / @submit.prevent —— 模板里直接写
```

#### 3.3.12 Map/Set/WeakMap

```javascript
const m = new Map([['a', 1]])
m.set('b', 2); m.get('a'); m.has('b'); m.delete('a')
// Map vs 普通对象：任意类型做键、有序、有 size、频繁增删性能更好、可直接 for...of 遍历

const s = new Set([1, 2, 2, 3])   // {1,2,3} —— 数组去重一行：[...new Set(arr)]
s.add(4); s.has(1); s.delete(2)

const wm = new WeakMap()          // 键只能是对象，且是弱引用（键无其他引用时可被 GC）
// Vue 响应式系统内部用 WeakMap 存"对象→依赖"，防止被框架持有的映射阻止对象回收 —— 见阶段 5
```

### 3.4 阶段 1 自查题 + 优秀解答

**Q1：手写防抖与节流，说明区别与场景。**

> **防抖**：事件停止触发 delay 毫秒后才执行一次（期间再触发则重新计时）——**只要还在动就不执行**。适合：搜索输入联想、窗口 resize 结束后计算、表单停止输入后校验。
> **节流**：固定间隔内最多执行一次——**再怎么频繁也按时执行**。适合：滚动加载、按钮防连点、拖拽。

```typescript
// 防抖（带立即执行选项）
function debounce<T extends (...args: any[]) => void>(fn: T, delay = 300, immediate = false) {
  let timer: ReturnType<typeof setTimeout> | null = null
  return function (this: unknown, ...args: Parameters<T>) {
    if (timer) clearTimeout(timer)
    if (immediate && !timer) fn.apply(this, args)   // 第一次立即执行
    timer = setTimeout(() => {
      timer = null
      if (!immediate) fn.apply(this, args)
    }, delay)
  }
}

// 节流（时间戳版：首触发立即执行）
function throttle<T extends (...args: any[]) => void>(fn: T, interval = 300) {
  let last = 0
  return function (this: unknown, ...args: Parameters<T>) {
    const now = Date.now()
    if (now - last >= interval) { last = now; fn.apply(this, args) }
  }
}
```

**Q2：Promise 三状态？`all/allSettled/race/any` 区别？**

> 见 3.3.8 完整答案。一句话总结：`all` 一损俱损；`allSettled` 全都要结果；`race` 先到先得（含失败）；`any` 第一个成功的。

**Q3：`map` 与 `forEach` 区别？用 `reduce` 实现 `map`？**

> `map` 返回新数组且不改变原数组（函数式，可链式）；`forEach` 无返回值、只为副作用。

```javascript
Array.prototype.myMap = function (fn) {
  return this.reduce((acc, cur, idx, arr) => {
    acc.push(fn(cur, idx, arr))
    return acc
  }, [])
}
```

**Q4：说出事件循环输出顺序并解释。** → 见 3.3.10，答题框架：“同步代码 → 每个宏任务后排空全部微任务 → 渲染 → 下一轮”。

**Q5：闭包是什么？两个应用与一个坑？** → 见 3.3.2，答题框架：“函数+词法环境的引用；应用=私有变量/防抖节流/Composable；坑=循环var与内存泄漏”。

---

## 四、阶段 2：TypeScript 系统学习（第 5~7 周）

> **先建立总认知**：TS = JS + 编译期类型系统。所有类型在**编译后全部擦除**，运行时就是纯 JS。你 Java/Kotlin 的泛型、接口思维直接复用，重点学“TS 独有玩法”。

### 4.1 类型标注基础

```typescript
// 基础类型（小写！string 不是 String）
let name: string = '张三'
let count: number = 0
let loading: boolean = false
let ids: number[] = [1, 2]
let pair: [string, number] = ['a', 1]          // 元组：定长定类型数组
let nothing: void = undefined                   // 函数无返回值
let u: undefined; let n: null

// 类型推断：能推断就不手写（代码更干净）
let age = 18            // 推断为 number（不是 any）
const site = 'cn'       // 推断为字面量类型 'cn'

// 联合类型：值可能是多种之一 —— TS 表达"多态"的日常手段
let status: 'idle' | 'loading' | 'success' | 'error' = 'idle'
let id: number | string = 123

// 数组与函数
const list: Array<{ id: number }> = []
function add(a: number, b: number): number { return a + b }
// 可选参数与默认值
function greet(name: string, greeting = '你好'): string { return `${greeting}, ${name}` }
// 剩余参数
function sum(...nums: number[]): number { return nums.reduce((a, b) => a + b, 0) }
```

### 4.2 interface vs type（面试必考）

```typescript
// interface：描述对象形状，支持声明合并、extends 继承
interface Animal { name: string }
interface Animal { age: number }          // ✔ 声明合并：两个同名自动合并（也给第三方库扩展留口子）
interface Dog extends Animal { breed: string }

// type：别名，能力更广 —— 联合、交叉、条件、映射类型都只有 type 能做
type ID = number | string
type Admin = User & { permissions: string[] }   // 交叉类型：合并多个类型的全部成员
type Status = 'active' | 'disabled'

// 共同点：都能描述对象；都能被 implements/extends
// 结论（社区惯例）：对象结构优先 interface，其他场景（联合/工具类型/映射）用 type
```

### 4.3 类型收窄 —— TS 从“能用”到“好用”的关键

TS 会根据代码逻辑动态推断 narrowing 后的类型：

```typescript
function format(value: string | number) {
  if (typeof value === 'string') {
    value.toUpperCase()   // 此分支内 value 已收窄为 string
  } else {
    value.toFixed(2)      // 此分支为 number
  }
}

// instanceof / in 收窄
if (error instanceof TypeError) { }
if ('id' in obj) { obj.id }

// ★ 判别式联合 —— 前端"状态建模"的标准范式（必须掌握）
// 多个类型共享一个字面量字段作为"标签"
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User[] }     // success 才有 data
  | { status: 'error'; message: string }    // error 才有 message

function render(state: RequestState) {
  switch (state.status) {          // 检查标签字段 → TS 自动收窄到对应分支
    case 'idle':    return '待请求'
    case 'loading': return '加载中'
    case 'success': return state.data.map(u => u.name).join(',')  // ✔ state.data 有类型
    case 'error':   return state.message                            // ✔
  }
}
// 对比：用一个 { status, data?, message? } 大杂烩类型，你永远不知道哪个状态该有哪个字段
```

### 4.4 泛型精讲

```typescript
// 泛型函数：T 是调用时才确定的类型参数
function identity<T>(value: T): T { return value }
identity<string>('a')   // 显式指定
identity(42)            // 推断为 identity<number>

// 泛型约束：限制 T 必须具备的能力
function getLength<T extends { length: number }>(item: T): number {
  return item.length                    // ✔ 因为约束保证有 length
}
getLength('abc'); getLength([1, 2])     // ✔
getLength(123)                          // ✘ number 没有 length

// keyof + 泛型：实现类型安全的属性访问（Vue 源码/工具库大量使用）
function prop<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
const user = { id: 1, name: '张三' }
prop(user, 'name')   // 返回类型自动推断为 string
prop(user, 'age')    // ✘ 编译期直接报错 —— 这就是类型安全的价值

// 泛型接口 + 默认类型参数
interface ApiResponse<T = unknown> {
  code: number
  message: string
  data: T
}
```

### 4.5 工具类型 + 手写实现（面试高频）

```typescript
interface User { id: number; name: string; age: number }

// Partial<T>：全部变可选 —— 更新接口的 DTO
type PartialUser = Partial<User>            // { id?: number; name?: string; age?: number }

// Required<T>：全部变必选（Partial 的反向）
// Readonly<T>：全部只读 —— 传入函数防修改
// Pick<T, K>：挑选部分字段 —— 列表页瘦身模型
type UserBrief = Pick<User, 'id' | 'name'>  // { id: number; name: string }

// Omit<T, K>：剔除字段 —— 创建接口去掉 id（后端生成）
type UserCreateDTO = Omit<User, 'id'>

// Record<K, V>：键值对映射 —— 字典/枚举翻译
type RoleLabel = Record<'admin' | 'editor' | 'viewer', string>
const labels: RoleLabel = { admin: '管理员', editor: '编辑', viewer: '访客' }

// ReturnType<T>：提取函数返回值类型
function fetchUser() { return { id: 1 } }
type User = ReturnType<typeof fetchUser>   // { id: number } —— API 函数不用重复定义返回类型

// Awaited<T>：提取 Promise 的值类型
type UserData = Awaited<ReturnType<typeof fetchUserAsync>>
```

**手写实现（理解“映射类型”语法）**：

```typescript
// [K in keyof T]：遍历 T 的每个键；?: 加可选修饰；T[K] 取对应值类型
type MyPartial<T> = { [K in keyof T]?: T[K] }

type MyPick<T, K extends keyof T> = { [P in K]: T[P] }

type MyReadonly<T> = { readonly [K in keyof T]: T[K] }

// 进阶：infer 在条件类型中"声明一个待推断的变量"
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never
// 读法：如果 T 能匹配"函数签名"，就把返回值位置捕获为 R，结果为 R；否则 never

type MyAwaited<T> = T extends Promise<infer V> ? V : T
```

### 4.6 any / unknown / never —— 三者的本质（面试必考）

```typescript
// any：放弃类型检查 —— 双向兼容一切，编译器完全躺平。禁用。
let a: any = 1; a.foo.bar()   // 编译不报错，运行时崩

// unknown：类型安全的 any —— 任何值可以赋给它（单向），但它不能赋给别的类型，
// 使用前必须先收窄/校验 —— 处理"来源不确定的数据"（接口响应、JSON.parse）的正解
let u: unknown = JSON.parse('{"a":1}')
u.a              // ✘ 编译报错：不能直接用
if (typeof u === 'object' && u !== null && 'a' in u) {
  console.log(u) // ✔ 收窄后使用
}

// never：不可能存在的类型
// ① 抛异常/死循环函数的返回类型  ② 收窄到"排除所有可能"后的剩余分支
function fail(): never { throw new Error('x') }

// 经典应用：never 做穷尽性检查 —— 漏处理一个 case 时编译报错
type Shape = Circle | Square
function area(s: Shape) {
  switch (s.kind) {
    case 'circle': return /* ... */ 0
    case 'square':  return /* ... */ 0
    default:
      const _exhaustive: never = s   // 若未来加了 Triangle 而忘了处理，这里立刻编译报错
      return _exhaustive
  }
}
```

### 4.7 类型断言、satisfies 与类型守卫

```typescript
// 断言：我比编译器更懂（慎用，是逃生舱不是常规手段）
const el = document.getElementById('app') as HTMLDivElement

// 双重断言（最后手段）：unknown as T

// satisfies（TS 4.9+）：校验类型但保留更精确的字面量推断 —— 现代推荐
const theme = {
  primary: '#409eff',
  danger: '#f56c6c',
} satisfies Record<string, string>
theme.primary   // 类型是 '#409eff' 字面量（用 as 或标注会丢失成 string）

// 自定义类型守卫：is 语法
function isUser(v: unknown): v is User {
  return typeof v === 'object' && v !== null && 'id' in v && 'name' in v
}
const data: unknown = await res.json()
if (isUser(data)) data.name   // ✔ 守卫通过后自动按 User 处理
```

### 4.8 tsconfig 关键配置

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",            // 编译目标 JS 版本
    "module": "ESNext",            // 产物模块格式（交给 Vite 处理）
    "moduleResolution": "bundler", // 按打包器规则解析模块（Vite 项目标配）
    "strict": true,                // ★ 严格模式全家桶，企业项目必开
    "noUncheckedIndexedAccess": true, // arr[i] 自动带 undefined，防越界
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }  // 路径别名，与 Vite 配置同步
  },
  "include": ["src/**/*.ts", "src/**/*.vue"]
}
```

**`.d.ts` 补充**：给没有类型的第三方 JS 库补声明；`vite-env.d.ts` 中 `/// <reference types="vite/client" />` 让 `import.meta.env` 有类型。

### 4.9 阶段 2 自查题 + 优秀解答

**Q1：手写 `MyPick` / `MyPartial`。** → 见 4.5 代码。答题要点：泛型约束 `K extends keyof T` + 映射类型 `[P in K]`。

**Q2：interface 与 type 区别？**

> ① interface 支持声明合并（同名自动合并），type 不支持；② type 能表达联合/交叉/条件/映射类型，interface 只能描述对象形状；③ extends 行为：interface 是“继承并检查兼容性”，交叉类型 `&` 是机械合并。惯例：对象结构用 interface，其余用 type。

**Q3：any/unknown/never 区别？** → 见 4.6。答题框架：“any 双向兼容放弃检查；unknown 只进不出、用前必收窄；never 不可能值，用于穷尽性检查”。

**Q4：用 TS 实现类型安全 EventEmitter。**

```typescript
interface EventMap {
  login: [userId: number, token: string]   // 事件名 → 参数元组
  logout: []
}

class EventEmitter<M extends Record<string, unknown[]>> {
  private handlers: { [K in keyof M]?: Set<(...args: M[K]) => void> } = {}

  on<K extends keyof M>(event: K, fn: (...args: M[K]) => void): () => void {
    ;(this.handlers[event] ??= new Set()).add(fn)
    return () => this.handlers[event]?.delete(fn)   // 返回解绑函数
  }

  emit<K extends keyof M>(event: K, ...args: M[K]): void {
    this.handlers[event]?.forEach(fn => fn(...args))
  }
}

const bus = new EventEmitter<EventMap>()
bus.on('login', (userId, token) => { })  // 参数类型自动推断 ✔
bus.emit('login', 1, 'abc')              // ✔
bus.emit('login', '1')                   // ✘ 编译期报错
```

**Q5：把阶段 1 的 debounce 改写为完整 TS 版。** → 见 3.4 Q1，要点：泛型约束函数签名、`this: unknown` 显式声明、`Parameters<T>` 拿参数元组、`ReturnType<typeof setTimeout>` 处理 Node/浏览器定时器类型差异。

---

## 五、阶段 3：Vue 3 核心技术（第 8~12 周，主战场）

### 5.1 响应式系统精讲（先懂原理，再看 API）

**Vue 3 响应式 = Proxy 代理 + 依赖收集 + 派发更新**。

**第一步：Proxy 如何拦截**（对比 Vue 2 的 `Object.defineProperty`）：

```javascript
const raw = { price: 100, qty: 2 }
const state = new Proxy(raw, {
  get(target, key, receiver) {
    console.log(`读取了 ${String(key)}`)         // 依赖收集的入口
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    const old = Reflect.get(target, key, receiver)
    const result = Reflect.set(target, key, value, receiver)
    if (old !== value) console.log(`${String(key)} 从 ${old} 变成 ${value}`)  // 派发更新入口
    return result
  },
})
state.price        // 打印"读取了 price"
state.price = 200  // 打印"price 从 100 变成 200"
```

**Proxy 相比 defineProperty 的三大优势**（面试标准答案）：
① 能监听**属性的增删**（defineProperty 无法监听后期新增的 key，Vue2 需要 `$set`）；② 能监听**数组索引和 length 修改**；③ 惰性深层代理——嵌套对象只有在被访问时才递归代理（Vue 2 初始化时递归遍历整棵树，大对象初始化开销大）。

**第二步：依赖收集的流程**（组件渲染的本质）：

```
组件渲染函数执行 → 被包在一个 effect（副作用）里运行
  → 模板读取 state.price → 触发 Proxy 的 get → track()
  → 把"当前正在运行的 effect"记录到该属性的依赖集合中
修改 state.price → 触发 set → trigger()
  → 找到该属性的依赖集合 → 重新运行其中的 effect → 组件重新渲染
```

组件初次渲染时“读了什么数据”，就决定了“之后什么数据变化会触发它更新”——**依赖是自动收集的，这就是声明式 UI 的底层机制**（等价于 Compose 的快照读追踪）。

### 5.2 ref 与 reactive —— 用法与所有坑

```vue
<script setup lang="ts">
import { ref, reactive, computed, toRefs } from 'vue'

// ref：包装任意类型（基本类型必须用 ref），读写需 .value，模板中自动解包
const count = ref(0)              // Ref<number>，类型泛型可显式：ref<number | null>(null)
count.value++                     // script 中必须 .value

// reactive：仅限对象/数组/Map/Set，直接属性访问
const form = reactive({ name: '', age: 18 })
form.name = '张三'                 // 无需 .value

// computed：基于依赖缓存，依赖不变不重算
const intro = computed(() => `${form.name}，${form.age} 岁`)
</script>

<template>
  <!-- 模板中 ref 自动解包，不写 .value -->
  <button @click="count++">点击 {{ count }} 次</button>
  <p>{{ intro }}</p>
</template>
```

**reactive 的三大坑（面试 + 实战高频）**：

```typescript
const state = reactive({ user: { name: '张三' }, tags: ['a'] })

// 坑1：解构丢失响应性 —— 解构把属性值"拷出来"，脱离了 Proxy 的追踪
let { user } = state          // user 之后不再响应 state.user 的变化
// 正确姿势：toRefs 把每个属性转成 ref（指向原属性）
const { user, tags } = toRefs(state)   // user 是 Ref，模板中照常用，仍保持响应

// 坑2：整体替换丢失响应性
// state = reactive({...newData})   ✘ 变量指向了新对象，旧的 Proxy 被抛弃
Object.assign(state, newData)  // ✔ 在原 Proxy 上逐属性赋值

// 坑3： reactive(new Date()) 等内置对象 / 传入 ref 会被解包等边界行为

// 实践结论：默认全用 ref（统一 .value 心智，避免三大坑），仅"表单等聚合对象"用 reactive
```

**为什么 ref 需要 .value**（原理面试题）：基本类型（number 等）无法被 Proxy 代理，Vue 把它装进一个带 `value` 属性的“盒子对象”再代理盒子——`.value` 的 get/set 就是依赖收集/派发的触发点。模板中编译器自动插入 `.value`，所以你不用写。

**深层响应与性能**：`ref({a:{b:1}})` 默认深层响应（深层修改也触发更新）；大列表等场景用 `shallowRef` 只监听 `.value` 整体替换，配合 `triggerRef()` 手动触发。

### 5.3 computed / watch / watchEffect 全对比（面试必考）

```typescript
// computed：有缓存的派生值。多次读取不重算（依赖没变时直接返回缓存）；
// 不应有副作用（不要在里面发请求/改其他状态）
const fullName = computed(() => `${firstName.value} ${lastName.value}`)
// 可写 computed（双向绑定中间层）
const proxyName = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})

// watch：明确指定"监听谁"，懒执行（源变化才触发），能拿到新旧值
watch(count, (newVal, oldVal) => {           // 监听 ref
  console.log(`${oldVal} → ${newVal}`)
})
watch(() => user.age, (n, o) => {})          // 监听 reactive 的某属性：getter 函数
watch(user, () => {}, { deep: true })        // reactive 整体（默认已深层）
watch([count, () => user.age], ([n1, n2]) => {})  // 多源
watch(count, cb, { immediate: true, deep: false, flush: 'pre' })
// immediate：创建时立即执行一次（初始化逻辑）
// flush：'pre'组件更新前(默认) / 'post'组件更新后(要拿更新后DOM) / 'sync'同步

// watchEffect：立即执行一次，执行中"读到谁就自动监听谁"，拿不到旧值
watchEffect(() => {
  // 立即执行；内部读取了 userId.value → 自动建立依赖
  fetchProfile(userId.value)
})
// 适合："副作用与依赖关系一目了然、不关心旧值" 的场景，如自动同步 localStorage
// 清理副作用（防止竞态：旧请求回来覆盖新数据）
watchEffect(async (onCleanup) => {
  const ctrl = new AbortController()
  fetchProfile(userId.value, ctrl)
  onCleanup(() => ctrl.abort())   // 下次触发前 / 停止时调用 → 取消上一次
})
```

**选择口诀**：**算出来的值用 computed；要旧值/明确目标用 watch；自动追踪副作用用 watchEffect**。

### 5.4 模板语法与指令全解

```vue
<template>
  <!-- 文本插值 -->
  <span>{{ message }}</span>
  <!-- 属性绑定：v-bind 缩写 : -->
  <img :src="avatar" :alt="`${name}的头像`" />
  <!-- 动态类名/样式的对象语法（高频） -->
  <div :class="{ active: isActive, disabled: isDisabled }" :style="{ color: textColor }"></div>
  <!-- 事件：v-on 缩写 @；修饰符 -->
  <form @submit.prevent="onSubmit">        <!-- .prevent = preventDefault -->
    <button @click.stop="doSomething">     <!-- .stop = stopPropagation -->
    <input @keyup.enter="query" />         <!-- 按键修饰符：回车触发 -->
    <a @click.once="showTip">              <!-- 只触发一次 -->
    <div @click.capture / @click.self>     <!-- 捕获阶段 / 仅自身触发（不含子元素冒泡） -->
  </form>

  <!-- 条件渲染 -->
  <p v-if="score >= 90">优秀</p>           <!-- 真正的创建/销毁 DOM，切换开销大 -->
  <p v-else-if="score >= 60">及格</p>
  <p v-else>不及格</p>
  <p v-show="isTipVisible">提示</p>        <!-- 仅 display:none 切换，初始开销大 -->

  <!-- 列表渲染：key 必须是稳定唯一标识（数据库 id），禁止用 index -->
  <li v-for="item in items" :key="item.id">
    {{ item.name }} <span v-if="item.done">✓</span>
  </li>
  <!-- v-for + v-if 不要同标签：v-for 优先级更高（Vue3），应 computed 先过滤 -->

  <!-- v-model 表单双向绑定（本质 :value + @input 的语法糖） -->
  <input v-model="text" />
  <input v-model.number="age" />   <!-- 自动转数字 -->
  <input v-model.trim="name" />    <!-- 去首尾空格 -->
  <select v-model="selected" />
  <input type="checkbox" v-model="checkedList" value="a" />  <!-- 多选绑数组 -->

  <!-- v-html：渲染富文本 —— 有 XSS 风险，内容必须可信/消毒 -->
  <div v-html="richText"></div>
</template>
```

**v-if vs v-show 决策**：切换频繁（hover 提示、Tab 内容）→ `v-show`；条件很少变化（权限控制、错误页）→ `v-if`。

**key 的原理（面试必答）**：diff 复用判断的依据。用 `index` 作 key，在**头部插入/删除**节点时，后续所有节点 key 全部错位，Vue 会就地更新每个节点的内容而不是移动节点——不仅浪费，还会导致**表单类组件状态错乱**（输入框内容残留）。用稳定 id 则能精准识别“哪些节点只是移动了位置”。

### 5.5 生命周期详解

```vue
<script setup lang="ts">
import { onMounted, onUpdated, onUnmounted, onActivated, onDeactivated } from 'vue'

// setup() 本身 ≈ beforeCreate + created（此时还没挂载，拿不到 DOM）
// 场景：初始化响应式数据、注册 composable

onMounted(() => {
  // DOM 已渲染完毕。场景：请求数据、初始化图表/地图实例、addEventListener、
  // 通过 ref 访问子组件/DOM
})

onUpdated(() => { /* 响应式状态变化导致 DOM 更新后。少用，易造成循环更新 */ })

onUnmounted(() => {
  // 组件销毁。必须清理：clearInterval(timer)、removeEventListener、
  // AbortController.abort() —— 不清理 = 内存泄漏 + 幽灵回调（高频线上事故来源）
})

// 配合 <KeepAlive> 的缓存专用钩子
onActivated(() => {})     // 缓存组件被重新激活（每次进入页面都会触发，区别于 mounted 只触发一次）
onDeactivated(() => {})
</script>
```

**执行顺序全景**：`setup` → 渲染 → `onMounted`（子组件先于父组件 mounted）→ 数据变化 → `onUpdated` → 卸载 `onUnmounted`。
**请求数据放 setup 还是 onMounted？** Composition API 下两者皆可（setup 直接触发的请求不等 DOM），惯例写在 setup 顶层或 onMounted，团队统一即可。

### 5.6 组件化与通信（六种方式全配代码）

**Props 与 Emits（父子通信，90% 场景）**：

```vue
<!-- UserCard.vue -->
<script setup lang="ts">
// ★ Vue 3.5+：响应式 props 可以直接解构（编译器自动转为 props.x），不再丢失响应性
interface Props {
  user: User
  disabled?: boolean        // 可选 prop
}
const { user, disabled = false } = defineProps<Props>()

// ★ 3.3+ 具名元组语法：事件名与参数全部类型安全
const emit = defineEmits<{
  select: [id: number]
  remove: [user: User]
}>()
</script>

<template>
  <div class="card" :class="{ disabled }" @click="!disabled && emit('select', user.id)">
    <img :src="user.avatar" />
    <span>{{ user.name }}</span>
    <button @click.stop="emit('remove', user)">删除</button>
  </div>
</template>
```

```vue
<!-- 父组件使用 -->
<script setup lang="ts">
const users = ref<User[]>([])
const onSelect = (id: number) => console.log('选中', id)
</script>
<template>
  <UserCard
    v-for="u in users" :key="u.id"
    :user="u"
    :disabled="u.id === lockedId"
    @select="onSelect"
    @remove="users = users.filter(x => x.id !== $event.id)"
  />
</template>
```

**单向数据流**（面试必考）：props 只能父→子，子组件**禁止直接修改 props**。需要“改”时 emit 事件让父组件改，或用下面的 `v-model` 模式。原因：杜绝多来源修改导致数据流向不可追溯。

**组件 v-model 与 defineModel（3.4+，现代必会）**：

```vue
<!-- 自定义输入组件 CustomInput.vue -->
<script setup lang="ts">
// 一行搞定双向绑定：编译为 :modelValue + @update:modelValue
const model = defineModel<string>({ required: true })
</script>
<template>
  <input :value="model" @input="model = ($event.target as HTMLInputElement).value" />
</template>

<!-- 支持多个 v-model、带修饰符 -->
<script setup lang="ts">
const title = defineModel<string>('title')
const [size, sizeModifiers] = defineModel<string>('size', {
  set(v) { return sizeModifiers.capitalize ? v.toUpperCase() : v },
})
</script>

<!-- 父组件使用：等价于 :model-value="text" + @update:model-value="text = $event" -->
<CustomInput v-model="text" />
```

**插槽（内容分发）**：

```vue
<!-- Card.vue 布局组件 -->
<template>
  <div class="card">
    <header><slot name="header" :title="title">{{ title }}</slot></header>
    <main><slot /></main>   <!-- 默认插槽 -->
  </div>
</template>

<!-- 父组件：作用域插槽接收子组件传出的数据（类比 RecyclerView 多 ViewType） -->
<Card>
  <template #header="{ title }">
    <h3 class="custom">{{ title }} - 定制头部</h3>
  </template>
  <p>正文内容</p>
</Card>
```

**provide / inject（跨层级注入）**：

```typescript
// 类型安全的 key（官方推荐模式）
import { provide, inject, ref, readonly, type InjectionKey, type Ref } from 'vue'

const ThemeKey: InjectionKey<Ref<'light' | 'dark'>> = Symbol('theme')

// 祖先组件
const theme = ref<'light' | 'dark'>('light')
provide(ThemeKey, readonly(theme))   // readonly 防止后代篡改，单向数据流贯彻到底

// 任意后代组件
const theme = inject(ThemeKey)!      // 拿到 Ref，类型完整
theme.value                          // 只读访问；要修改应 provide 一个 changeTheme 方法
```

**模板引用（拿子组件实例/DOM）**：

```vue
<script setup lang="ts">
// Vue 3.5+ 推荐 useTemplateRef
import { useTemplateRef, onMounted } from 'vue'
const inputRef = useTemplateRef<HTMLInputElement>('input')   // 参数 = 模板中 ref 名
const formRef = useTemplateRef<FormInstance>('form')         // Element Plus 表单实例

onMounted(() => {
  inputRef.value?.focus()
})

// 调用子组件方法 —— 子组件必须 defineExpose 显式暴露
// 子组件内：defineExpose({ validate, resetFields })
const submit = () => formRef.value?.validate().then(/* ... */)
</script>
<template>
  <input ref="input" />
  <el-form ref="form">...</el-form>
</template>
```

**attrs 透传**：`useAttrs()` 拿到未被 props 声明的属性与事件（封装第三方组件二次封装时的“属性搬运”）。

**通信方式选型表（面试背熟）**：

| 方式 | 方向 | 场景 |
|---|---|---|
| props / emits | 父⇄子 | 90% 的组件通信 |
| v-model / defineModel | 父⇄子 | 表单、弹窗 visible |
| 插槽 / 作用域插槽 | 父→子内容 | 布局组件、列表定制 |
| provide / inject | 祖先→后代 | 主题、国际化、表单上下文 |
| Pinia | 全局 | 跨页面共享状态 |
| 模板引用 + defineExpose | 父→子方法 | 触发子组件校验/重置 |

### 5.7 完整实战：TodoMVC（基础功验收项目）

```vue
<!-- src/components/TodoApp.vue -->
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

interface Todo { id: number; text: string; done: boolean }
type Filter = 'all' | 'active' | 'done'

const todos = ref<Todo[]>([])
const newTodoText = ref('')
const filter = ref<Filter>('all')

// 派生：过滤后的列表（v-for 的数据源用 computed 而不是在模板里写过滤——可缓存、可测试）
const filteredTodos = computed(() => {
  const map = { all: () => true, active: (t: Todo) => !t.done, done: (t: Todo) => t.done }
  return todos.value.filter(map[filter.value])
})
const remaining = computed(() => todos.value.filter(t => !t.done).length)
const allDone = computed({
  get: () => remaining.value === 0 && todos.value.length > 0,
  set: (val) => todos.value.forEach(t => (t.done = val)),   // 可写 computed 实现“全选”
})

function addTodo() {
  const text = newTodoText.value.trim()
  if (!text) return
  todos.value.push({ id: Date.now(), text, done: false })   // ref 数组直接 push（深层响应）
  newTodoText.value = ''
}
function removeTodo(id: number) {
  todos.value = todos.value.filter(t => t.id !== id)        // 不可变风格
}

// 持久化：watch 深层监听 + localStorage
watch(todos, (val) => localStorage.setItem('todos', JSON.stringify(val)), { deep: true })
onMounted(() => {
  todos.value = JSON.parse(localStorage.getItem('todos') ?? '[]')
})
</script>

<template>
  <div class="todo">
    <input v-model="newTodoText" @keyup.enter="addTodo" placeholder="输入待办，回车添加" />
    <label><input type="checkbox" v-model="allDone" /> 全选</label>
    <select v-model="filter">
      <option v-for="f in (['all','active','done'] as Filter[])" :key="f" :value="f">{{ f }}</option>
    </select>
    <ul>
      <li v-for="todo in filteredTodos" :key="todo.id">
        <input type="checkbox" v-model="todo.done" />
        <span :class="{ done: todo.done }">{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)">×</button>
      </li>
    </ul>
    <p>剩余 {{ remaining }} 项</p>
  </div>
</template>

<style scoped>
.done { text-decoration: line-through; color: #999; }
</style>
```

**逐段复盘要点**（做完必须能回答）：① 为什么 `filteredTodos` 用 computed 而不是方法？——缓存 + 依赖不变不重算；② `allDone` 为什么需要 get/set？——checkbox 双向绑定需要写回逻辑；③ `todos.value.push` 为什么能触发更新？——ref 深层响应，深层 Proxy 拦截。

### 5.8 组合式函数 Composables（初级→中级的分水岭）

**规范**：命名 `useXxx`；只在 `setup`/组合式函数中调用；返回 `ref` 组成的对象（调用方解构仍保持响应）；内部完成清理。

```typescript
// useLocalStorage.ts —— 带类型的 localStorage 响应式同步
import { ref, watch, type Ref } from 'vue'

export function useLocalStorage<T>(key: string, initialValue: T) {
  const stored = localStorage.getItem(key)
  const data: Ref<T> = ref(stored ? (JSON.parse(stored) as T) : initialValue)

  watch(data, (val) => localStorage.setItem(key, JSON.stringify(val)), { deep: true })
  return data
}

// 用法：const token = useLocalStorage<string>('token', '')
```

```typescript
// useRequest.ts —— 请求全状态管理（含手动重试）
import { ref, shallowRef, type Ref } from 'vue'

export function useRequest<T>(fn: () => Promise<T>, options: { immediate?: boolean } = { immediate: true }) {
  const data = shallowRef<T | null>(null) as Ref<T | null>  // shallowRef：data 整体替换，避免深层代理大对象开销
  const loading = ref(false)
  const error = ref<Error | null>(null)

  async function run() {
    loading.value = true; error.value = null
    try { data.value = await fn() }
    catch (e) { error.value = e as Error }
    finally { loading.value = false }
  }
  if (options.immediate) run()
  return { data, loading, error, run }
}

// 页面中三行接入：
const { data: users, loading, run: refetch } = useRequest(() => userApi.getList())
```

```typescript
// useCountDown.ts —— 倒计时（展示"必须在 onUnmounted 清理"）
import { ref, onUnmounted } from 'vue'

export function useCountDown(seconds: number) {
  const remaining = ref(seconds)
  const timer = setInterval(() => {
    remaining.value--
    if (remaining.value <= 0) clearInterval(timer)
  }, 1000)
  onUnmounted(() => clearInterval(timer))   // ★ 不写这行：页面离开后定时器仍跑（泄漏+幽灵更新）
  return { remaining }
}
```

**VueUse 必学清单**（先读源码再仿写）：`useMouse`、`useEventListener`（自动清理）、`useIntersectionObserver`（曝光埋点/无限滚动）、`useVirtualList`（万级列表）、`useDebounceFn`、`useClipboard`、`useDark`。

### 5.9 高级特性

**Teleport**：

```vue
<!-- 把弹窗 DOM 传送到 body —— 逃脱父级 overflow:hidden / z-index 层叠上下文/transform 裁剪 -->
<Teleport to="body">
  <div v-if="visible" class="modal">...</div>
</Teleport>
```

**KeepAlive**：

```vue
<!-- 缓存组件实例：列表页→详情页→返回，列表滚动位置与筛选条件全保留 -->
<RouterView v-slot="{ Component }">
  <KeepAlive :include="['UserList', 'OrderList']">   <!-- 匹配组件的 name（script setup 用 defineOptions 定义） -->
    <component :is="Component" />
  </KeepAlive>
</RouterView>
```

**Suspense / 异步组件**（了解）：`defineAsyncComponent(() => import('./Heavy.vue'))` 实现按需加载与骨架屏协调。

**自定义指令**（了解 + 一个实用例子）：

```typescript
// v-permission：按钮级权限 —— 无权限直接从 DOM 移除
export const vPermission: Directive<HTMLElement, string[]> = {
  mounted(el, binding) {
    const roles = useUserStore().roles
    if (!binding.value.some(r => roles.includes(r))) el.parentNode?.removeChild(el)
  },
}
// 使用：<button v-permission="['admin']">删除</button>
```

**nextTick 原理（面试必考）**：修改响应式数据后，Vue 的 DOM 更新**不是同步的**——组件更新函数被推入队列，在**微任务**中批量刷新（同一轮多次修改只更新一次，性能优化）。`nextTick(cb)` 把 cb 排到“队列刷新之后”执行，所以 cb 中能拿到最新 DOM。

```typescript
const list = ref<string[]>([])
async function addItem() {
  list.value.push('新项')
  console.log(document.querySelectorAll('li').length)  // 旧值（DOM 未更新）
  await nextTick()
  console.log(document.querySelectorAll('li').length)  // 新值
}
```

### 5.10 阶段 3 自查题 + 优秀解答

**Q1：ref 与 reactive 区别？为什么 reactive 解构会失效？**

> ref 包装任意类型，读写需 `.value`，重新赋值不丢响应性；reactive 仅限对象，直接属性访问，但解构/整体替换会失效。**失效原理**：解构是把属性值从 Proxy 对象上“取出来赋给普通变量”，后续修改变量只是一个普通赋值，不再经过 Proxy 的 set 拦截，自然无法触发更新。解决：`toRefs()` 把每个属性转为指向原始属性的 ref。

**Q2：手写 useDebounceRef（值防抖更新）。**

```typescript
import { customRef, type Ref } from 'vue'

// customRef：自己控制 track/trigger 时机 —— 手写响应式的练习场
export function useDebouncedRef<T>(value: T, delay = 300): Ref<T> {
  let timer: ReturnType<typeof setTimeout> | null = null
  return customRef<T>((track, trigger) => ({
    get() { track(); return value },                 // 读取时收集依赖
    set(newValue: T) {                               // 写入时延迟触发更新
      if (timer) clearTimeout(timer)
      timer = setTimeout(() => { value = newValue; trigger() }, delay)
    },
  }))
}
```

**Q3：computed 与 method 的区别？computed 与 watch 的区别？**

> computed 有缓存（依赖不变直接返回旧值），method 每次调用都执行；computed 用于派生数据，watch 用于响应变化执行副作用（请求、持久化、DOM 操作）。computed 不应包含副作用。

**Q4：v-if 与 v-show？key 为什么不能用 index？** → 见 5.4，答题要点：渲染机制差异 + diff 复用错位 + 表单状态残留案例。

**Q5：独立完成 TodoMVC 并能逐段讲清原理**（讲不清的地方回到 5.7 复盘要点重学）。

---

## 六、阶段 4：工程化与生态（第 13~16 周）

### 6.1 Vue Router 4 深入

```typescript
// src/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/login',
    name: 'login',
    component: () => import('@/views/Login.vue'),   // ★ 路由懒加载：切到该页才下载代码
  },                                                 // （首屏体积优化的第一手段）
  {
    path: '/',
    component: () => import('@/layouts/BasicLayout.vue'),
    meta: { requiresAuth: true, title: '首页' },    // meta：路由元信息（自定义字段）
    children: [
      { path: '', name: 'home', component: () => import('@/views/Home.vue') },
      // 动态路由：路径参数
      { path: 'user/:id(\\d+)', name: 'user-detail',
        component: () => import('@/views/UserDetail.vue'),
        props: true },   // ★ props:true 把 params 作为 props 传入 → 组件与路由解耦（可测试）
    ],
  },
  { path: '/:pathMatch(.*)*', name: 'not-found', component: () => import('@/views/404.vue') },
]

const router = createRouter({ history: createWebHistory(), routes })

// ===== 导航守卫：完整执行顺序（面试必背）=====
// 1. 组件内 beforeRouteLeave（离开当前页）
// 2. 全局 beforeEach
// 3. 路由独享 beforeEnter
// 4. 组件内 beforeRouteEnter（拿不到 this/实例）
// 5. 全局 beforeResolve（所有守卫/异步组件解析完成）
// 6. 导航确认 → 全局 afterEach（可在此改 document.title）
// 7. DOM 更新 → beforeRouteEnter 的 next(vm => {}) 回调（此时可访问实例）

router.beforeEach(async (to, _from) => {
  const userStore = useUserStore()
  // 返回值语义：false=取消导航 / 路由对象=重定向 / undefined=true=放行（Vue3 不再强制 next）
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    return { name: 'login', query: { redirect: to.fullPath } }   // 记录回跳地址
  }
})

router.afterEach((to) => { document.title = `${String(to.meta.title ?? '')} - 管理系统` })
```

```typescript
// 组件内获取参数与编程式导航
import { useRoute, useRouter } from 'vue-router'
const route = useRoute()      // 当前路由信息（path/params/query/meta）
const router = useRouter()
route.params.id                                   // 声明式：模板中 $route.params.id
router.push('/users')                             // 跳转（加历史记录，可后退）
router.replace('/login')                          // 替换（不可后退）—— 登录后常用
router.back()                                     // 返回
// 监听同一个路由组件的参数变化（/user/1 → /user/2 组件被复用，不会重新创建！）
watch(() => route.params.id, (newId) => fetchData(newId))
```

**history vs hash 模式**：`createWebHistory()`（`/user/1`，需服务端配置所有路径回退到 index.html）vs `createWebHashHistory()`（`/#/user/1`，免配置但丑）。企业中后台用 history + Nginx `try_files $uri $uri/ /index.html;`。

**RBAC 动态路由（项目三核心功能）**：登录后请求后端菜单 → 过滤出本角色可见的 `RouteRecordRaw` → `router.addRoute(route)` 逐条注册 → `addRoute` 后必须 `router.replace(to.fullPath)` 重新触发导航。

### 6.2 Pinia 深入

```typescript
// src/stores/user.ts —— Setup Store 写法（与 Composition API 心智统一，官方推荐）
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // ============ state ============
  const token = ref('')
  const userInfo = ref<UserInfo | null>(null)
  // ============ getters ============
  const isLoggedIn = computed(() => !!token.value)
  const isAdmin = computed(() => userInfo.value?.role === 'admin')
  // ============ actions（可异步） ============
  async function login(payload: LoginParams) {
    const { token: t, user } = await authApi.login(payload)
    token.value = t
    userInfo.value = user
  }
  function logout() { token.value = ''; userInfo.value = null; router.push('/login') }
  // ★ 异步 action 之间可互相调用：直接调函数名即可，无 Vuex 的 commit/dispatch 样板
  return { token, userInfo, isLoggedIn, isAdmin, login, logout }
})
```

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// ★ 核心规则：state 和 getter 是响应式数据，解构必须 storeToRefs（原理同 reactive 解构）
const { token, isLoggedIn } = storeToRefs(userStore)
// action 是普通函数，直接解构安全
const { logout } = userStore

// 修改 state 的三种方式：
userStore.token = 'xxx'                     // 1. 直接改
userStore.$patch({ token: 'xxx', userInfo: u })  // 2. patch 批量改
// 3. 调用 action（含业务逻辑时唯一正解）
</script>
```

**持久化**：`pinia-plugin-persistedstate` 插件，Store 上配置 `persist: true`（或指定 paths）。**跨 Store 引用**：在一个 store 的 action 里直接 `useCartStore()` 调用。

**什么时候用 Pinia 什么时候用组件内状态？** 判据：数据是否需要**跨越页面/跨越组件树共享**（用户信息、购物车、主题）。页面私有数据（表单临时值）留在组件内。

### 6.3 Axios 封装（企业必考，逐段讲解）

```typescript
// src/utils/request.ts
import axios, { type AxiosInstance, type InternalAxiosRequestConfig } from 'axios'
import { useUserStore } from '@/stores/user'

// ===== 第一层：实例与基础配置 =====
const service: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,   // 环境变量（.env.development / .env.production）
  timeout: 10_000,
})

// ===== 第二层：请求拦截 —— 统一注入凭证 =====
service.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// ===== 第三层：响应拦截 —— 抹平后端协议差异，只把 data 类型安全地返回 =====
service.interceptors.response.use(
  (response) => {
    const { code, message, data } = response.data    // 假设后端统一 { code, message, data }
    if (code !== 0) {
      // 业务错误：统一 toast，业务层无需再 try/catch
      ElMessage.error(message)
      return Promise.reject(new Error(message))
    }
    return data        // ★ 关键：调用方拿到的直接就是 data，不用每次 .data.data
  },
  (error) => {
    // HTTP 层错误分类处理
    if (error.response?.status === 401) {
      useUserStore().logout()          // 登录失效：清凭证 + 跳登录
    } else if (!window.navigator.onLine) {
      ElMessage.error('网络已断开')
    } else {
      ElMessage.error(error.response?.data?.message ?? '服务器开小差了')
    }
    return Promise.reject(error)
  }
)
export default service
```

```typescript
// src/api/user.ts —— 接口按业务模块拆分，全程类型安全
import request from '@/utils/request'

// 泛型第二个参数技巧：axios 的 get<T, R> 中 R 是拦截器处理后的返回类型
export const userApi = {
  getList: (params: PageParams) =>
    request.get<never, PageResult<User>>('/users', { params }),   // params 自动拼 query
  getById: (id: number) => request.get<never, User>(`/users/${id}`),
  create: (data: Omit<User, 'id'>) => request.post<never, User>('/users', data),
  update: (id: number, data: Partial<User>) => request.put<never, User>(`/users/${id}`, data),
  remove: (id: number) => request.delete<never, void>(`/users/${id}`),
}
```

**Token 无感刷新（进阶，面试加分）**：响应拦截器捕获 401 → 挂起当前请求 → 用 refreshToken 换新 token → 重发原请求 → 放行。**并发多个 401 时必须只刷新一次**：用一个 `isRefreshing` 标志 + 等待队列（Promise 数组），刷新完成后依次重发队列中的请求。

### 6.4 Vite 工程配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    vue(),
    // Element Plus 按需自动引入：模板里直接用 <el-button>，无需 import 和全量样式
    AutoImport({ resolvers: [ElementPlusResolver()] }),
    Components({ resolvers: [ElementPlusResolver()] }),
  ],
  resolve: { alias: { '@': '/src' } },
  server: {
    port: 5173,
    // 开发期跨域代理：前端请求 /api/users → Vite 转发到 target（浏览器全程同源，无跨域）
    proxy: {
      '/api': { target: 'http://localhost:3000', changeOrigin: true },
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {          // 手动分包：第三方依赖独立成 chunk，利用浏览器缓存
          if (id.includes('node_modules')) return 'vendor'
          if (id.includes('echarts')) return 'echarts'   // 大库单独拆
        },
      },
    },
  },
})
```

**环境变量**：`.env.development`（`VITE_API_BASE_URL=/api`）、`.env.production`（正式域名）。代码中 `import.meta.env.VITE_API_BASE_URL`。**必须以 `VITE_` 开头才会暴露给客户端**。

### 6.5 代码规范链路

```bash
pnpm add -D eslint prettier eslint-plugin-vue @vue/eslint-config-typescript \
  husky lint-staged commitlint @commitlint/config-conventional
```

流程：ESLint（9.x Flat Config，语法与风格问题）+ Prettier（纯格式化，二者分工）→ `husky` 注册 git hooks → `lint-staged` 只对暂存文件执行 → `commitlint` 校验提交信息格式（`feat: xxx` / `fix: xxx`，可自动生成 Changelog）。

**实操任务（阶段 4 总验收）**：把 TodoMVC 重构为多页面应用——Router 管页面 + Pinia 管全局待办 + Axios 对接 JSONPlaceholder + Element Plus 表格渲染 + 登录守卫。

---

## 七、阶段 5：原理、性能与浏览器网络（第 17~22 周）★ 中高级门票

### 7.1 手写迷你响应式系统（50 行，面试高频手写题）

```typescript
// mini-reactivity.ts —— 覆盖 reactive/effect/track/trigger 核心机制
type EffectFn = (() => void) & { deps: Set<Set<EffectFn>> }

let activeEffect: EffectFn | null = null
// 依赖存储结构：WeakMap<目标对象, Map<属性, Set<副作用函数>>>
const targetMap = new WeakMap<object, Map<string | symbol, Set<EffectFn>>>()

export function reactive<T extends object>(obj: T): T {
  return new Proxy(obj, {
    get(target, key, receiver) {
      track(target, key)                       // 读取时：收集当前副作用
      const res = Reflect.get(target, key, receiver)
      // 真实源码这里还有：嵌套对象递归 reactive(res)、ref 解包
      return res
    },
    set(target, key, value, receiver) {
      const oldVal = Reflect.get(target, key, receiver)
      const result = Reflect.set(target, key, value, receiver)
      if (oldVal !== value) trigger(target, key)   // 值真的变了才触发（避免 NaN 等误触发）
      return result
    },
  })
}

function track(target: object, key: string | symbol) {
  if (!activeEffect) return                    // 不在 effect 中运行 → 无需收集
  let depsMap = targetMap.get(target)
  if (!depsMap) targetMap.set(target, (depsMap = new Map()))
  let dep = depsMap.get(key)
  if (!dep) depsMap.set(key, (dep = new Set()))
  dep.add(activeEffect)                        // 属性 → 副函数 建立映射
  activeEffect.deps.add(dep)                   // 反向记录（用于 cleanup）
}

function trigger(target: object, key: string | symbol) {
  const dep = targetMap.get(target)?.get(key)
  if (!dep) return
  // 拷贝后遍历：防止副作用执行中删除自己导致遍历混乱
  new Set(dep).forEach((fn) => fn())
}

export function effect(fn: () => void): EffectFn {
  const effectFn: EffectFn = (() => {
    activeEffect = effectFn
    cleanup(effectFn)      // 先清除旧依赖：处理“条件分支”导致依赖变化的场景
    fn()                    // 执行：fn 内读取的数据会重新完成 track
    activeEffect = null
  }) as EffectFn
  effectFn.deps = new Set()
  effectFn()
  return effectFn
}

function cleanup(effectFn: EffectFn) {
  effectFn.deps.forEach((dep) => dep.delete(effectFn))
  effectFn.deps.clear()
}

// ===== 验证：三行测试 =====
const state = reactive({ count: 0 })
effect(() => console.log('count 变为', state.count))  // 立即执行：count 变为 0
state.count = 1    // 自动重新执行：count 变为 1
state.count = 2    // count 变为 2
```

**对照真实源码的心智地图**：`effect(fn)` 就是组件渲染的包装（组件的 render 函数就是一个 effect）→ `computed` 是“带 dirty 缓存标记 + 不立即执行”的特化 effect → `watch` 是“指定源 + 用户回调”的 effect。**Vue 的更新调度是批量异步的**：trigger 不直接运行 effect，而是推入队列，在微任务中去重后统一执行——这是“一轮多次修改只渲染一次”的原因，也是 `nextTick` 存在的原因。

### 7.2 虚拟 DOM 与 diff 算法（面试必考，讲透）

**虚拟 DOM 是什么**：用 JS 对象描述 DOM 结构（`{ type: 'div', props: {...}, children: [...] }`）。**为什么需要**：直接操作 DOM 开销大，先用 JS 层面 diff 出**最小变更集**，再一次性应用，减少真实 DOM 操作；同时让渲染逻辑与平台解耦（浏览器 DOM / 移动端原生 / Canvas）。

**diff 的四大前提规则**：
① 只做**同层**比较（跨层级移动的节点直接销毁重建，不递归整棵树——保证复杂度 O(n)）；② 不同类型的节点（`div` → `span`）直接**不复用**，销毁重建；③ 通过 **key** 判断新旧节点是否为“同一个节点”；④ 静态内容跳过。

**key 的作用机制（举例讲透）**：旧列表 `[A, B, C]`，头部插入 X 变为 `[X, A, B, C]`：
- **用 index 作 key**：旧 `A(key=0)` ↔ 新 `X(key=0)`“配对成功”，于是 A 的内容被改成 X，B 改成 A……每个节点都被就地更新内容，且 C 后面新建一个——**全量更新，还导致表单组件内部状态错位**。
- **用 id 作 key**：diff 发现 `A/B/C` 的 key 在新旧两边都存在 → **只移动/新建 X，A/B/C 原样复用**。这就是“key 必须稳定唯一”的原理。

**Vue 3 的编译优化**（相对 Vue 2 提升性能的核心）：
- **Patch Flag**：编译期给动态节点打标记（`class 变化`/`文本变化`/`多个属性`），运行时 diff **只对比被标记的动态部分**，静态内容完全跳过。
- **Block Tree**：把动态节点按层级“拍平”收集，更新时沿数组遍历，跳过所有静态层级。
- **最长递增子序列**：处理带 key 的子节点列表移动时，求出“不需要移动的最长序列”，只移动序列外的节点，最小化 DOM 移动操作。

### 7.3 性能优化实战清单（每条附原理）

**加载优化（首屏）**：
| 手段 | 原理 |
|---|---|
| 路由懒加载 `() => import()` | 按路由切分 chunk，首屏只下载当前页代码 |
| 组件按需加载 | `defineAsyncComponent`，重弹窗/图表延迟加载 |
| `manualChunks` 分包 | vendor 独立 chunk，业务迭代时第三方库走浏览器缓存 |
| 图片：懒加载 + WebP + CDN + 合适尺寸 | 减少请求体积与阻塞 |
| Gzip/Brotli（Nginx 开启） | 传输体积降 70%+ |
| 预加载关键资源 `<link rel="preload">` | 提前拉取首屏必要资源 |

**运行时优化**：
| 手段 | 原理 |
|---|---|
| `v-show` 替代频繁切换的 `v-if` | 避免反复创建销毁组件 |
| `v-for` 用稳定 key + 用 computed 预过滤 | 避免 diff 错位 + 重复计算 |
| `shallowRef` 大列表 | 只监听整体替换，跳过深层 Proxy 开销 |
| **虚拟列表** | 只渲染可视区域 ± buffer 的节点，顶部 padding 撑总高；`scrollTop / itemHeight` 算起始索引。万级数据唯一正解（RecyclerView 同款思想，VueUse `useVirtualList`） |
| `KeepAlive` 缓存列表页 | 返回时免重新请求与渲染 |
| 防抖节流高频事件 | 见 3.4 |
| `v-memo="[item.id, item.selected]"` | 依赖不变跳过该项重渲染 |
| 长任务切片 | 拆成宏任务分帧执行，避免掉帧 |

**度量**：Lighthouse（LCP/CLS/INP 三大核心指标）、Performance 面板看长任务与渲染瀑布、`rollup-plugin-visualizer` 看包体积构成。

### 7.4 浏览器渲染与网络（中高级面试的重头戏）

#### 7.4.1 从输入 URL 到页面展示的完整过程（标准答题版）

```
1. URL 解析 → 2. DNS 解析（域名→IP，逐级缓存：浏览器→系统→路由器→运营商→递归查询）
→ 3. TCP 三次握手 → 4. TLS 握手（HTTPS）→ 5. 发送 HTTP 请求
→ 6. 服务器处理返回 HTML → 7. 浏览器解析渲染：
   ① 解析 HTML 构建 DOM 树（遇到 <script> 同步脚本会阻塞解析！defer/async 见下）
   ② 解析 CSS 构建 CSSOM
   ③ DOM + CSSOM 合成 Render Tree（不包含 display:none 的节点）
   ④ Layout 布局（计算几何信息） → ⑤ Paint 绘制 → ⑥ Composite 合成（GPU 分层）
→ 8. 异步加载剩余资源（图片/懒加载模块）
```

**defer vs async（script 加载）**：`defer`——并行下载，**DOM 解析完后、DOMContentLoaded 前按序执行**（保序，Vite 产物用的就是它）；`async`——并行下载，**下载完立即执行**（可能打断解析，不保序，适合独立统计脚本）。

#### 7.4.2 重排与重绘

- **重排**：几何属性变化触发，重新执行 Layout → Paint → Composite，代价大。
- **重绘**：仅外观变化（color/background），跳过 Layout。
- **合成**：仅 `transform / opacity`，直接在 GPU 合成层处理，**性能最好——动画首选这两个属性**。
- **避免强制同步布局**：JS 里“读布局属性（offsetTop/getBoundingClientRect）后紧接着写样式”，浏览器被迫同步重排。优化：先集中读、后集中写，或用 `requestAnimationFrame`。

#### 7.4.3 跨域（前端日常，必须彻底搞懂）

**同源策略**：协议 + 域名 + 端口**全相同**才同源。跨域限制的是 **JS 读取响应**，不是“发不出去请求”（请求其实到了服务器）。

**CORS（标准方案）**：
- **简单请求**：GET/POST/HEAD + 常规 Content-Type。浏览器直接发送，带 `Origin` 头；服务端返回 `Access-Control-Allow-Origin: https://a.com` 即可读。
- **预检请求**：满足任一条件时触发——方法为 PUT/DELETE/PATCH、Content-Type 为 `application/json`、带自定义 Header（如 `Authorization`）。浏览器先发一个 **OPTIONS** 请求询问，服务端须放行 `Access-Control-Allow-Methods/Headers`，通过后才发真实请求。**你的 Axios 封装携带 Token + JSON，每次跨域请求都伴随一次 OPTIONS**——这就是为什么后端不处理 OPTIONS 会导致“接口调不通但 Network 里看到的是 OPTIONS 报错”。
- **携带 Cookie**：前端 `axios` 设 `withCredentials: true`，后端 `Access-Control-Allow-Credentials: true` 且 `Allow-Origin` 不能是 `*`。

**开发期**：Vite proxy（见 6.4）——同源转发，无跨域。**生产期**：Nginx 反向代理（`location /api { proxy_pass http://backend; }`），前后端同域部署，一劳永逸。

#### 7.4.4 HTTP 缓存（强缓存 + 协商缓存，面试必考）

```
浏览器请求资源
  → 有缓存？→ 检查强缓存：
      Cache-Control: max-age=31536000 未过期 → 【直接用本地缓存，不发请求，200 (from disk/memory cache)】
      （Expires 是 HTTP/1.0 的绝对时间，已被 Cache-Control 取代）
  → 强缓存过期 → 发协商缓存请求（带条件头）：
      ETag 优先（内容 hash）: 请求头 If-None-Match: "abc123"
      Last-Modified（修改时间）: 请求头 If-Modified-Since: Wed, 21 Oct 2026 07:28:00 GMT
      → 资源没变：返回 304 Not Modified（无 body，几百字节）→ 用本地缓存
      → 变了：返回 200 + 新资源 + 新的 ETag
```

**实战要点**：带 hash 的静态资源（`app.a1b2c3.js`）设 `max-age=31536000, immutable` 永久强缓存——内容变则文件名变；`index.html` 设 `no-cache`（每次协商）——保证用户拿到最新入口从而加载新 hash 的资源。**“发布后用户看到的还是旧版本”九成是缓存策略配错**。

#### 7.4.5 认证：Session vs Token/JWT

- **Session**：状态存服务端，浏览器持 Cookie（SessionId）。分布式需要共享存储。**移动端开发者注意：Cookie 是浏览器自动携带的，App 原生请求没有这层自动机制**。
- **JWT**：状态存客户端（签名防篡改的 `header.payload.signature`）。前端存 localStorage（XSS 可窃取，需配合 CSP/消毒）vs 存 HttpOnly Cookie（JS 不可读，天然防 XSS，但 CSRF 需防护）。**企业主流实践：access token（短期）+ refresh token（长期）+ 401 无感刷新**。
- CSRF：恶意网站诱导用户浏览器自动带 Cookie 请求你的接口。防御：CSRF Token / SameSite Cookie / 敏感操作二次确认。

### 7.5 跨端与全栈延伸（选修）

- **Nuxt 4**：SSR（服务端渲染 HTML，首屏快 + SEO 友好）、文件路由、`useFetch`。理解 SPA vs SSR vs SSG 的取舍即可入门。
- **uni-app / Taro**：Vue 语法写小程序/App——**你的原生背景 + H5 经验 = 混合开发赛道的差异化竞争力**，转型过渡期的黄金岗位选择。
- **Node.js**：Koa/Express 写 CRUD + JWT 中间件，打通全栈认知闭环即可，不必深入。

---

## 八、阶段 6：项目实战路线（贯穿全程）

| 项目 | 阶段 | 技术要点 | 简历价值 |
|---|---|---|---|
| ① TodoMVC（TS 版） | 阶段 3 中 | 5.7 完整代码 | 练手，不上简历 |
| ② 移动端 H5 商城 | 阶段 4 | Vant 4、vw/rem 适配、路由、Pinia 购物车、图片懒加载、下拉刷新 | 发挥移动端存量经验 |
| ③ **中后台管理系统（主力）** | 阶段 4~5 | 见下方功能清单 | **求职核心项目** |
| ④ 全栈小项目 | 阶段 5 | Vue3 + Koa + MySQL + JWT | 工程闭环能力 |

**项目三功能清单（对标企业真实场景，逐项打勾）**：
- [ ] 登录（验证码 + JWT + 401 无感刷新 + 记住回跳）
- [ ] RBAC 动态路由（后端菜单 → `addRoute`）+ `v-permission` 按钮级权限
- [ ] 用户管理：表格 + 分页 + 多条件筛选 + 表单校验 + 批量删除
- [ ] Axios 统一封装（拦截器/错误分类/loading 防重复）
- [ ] 全局布局：面包屑、标签页缓存、主题切换（CSS 变量）、全屏
- [ ] ECharts 图表页 + 大数据虚拟列表页
- [ ] 路由级代码分包 + 图片懒加载 + Lighthouse 跑分 ≥ 85
- [ ] 全局错误上报（`app.config.errorHandler` + `window.onerror`）
- [ ] Docker 部署 + Nginx（history 回退 + Gzip + 缓存策略）

---

## 九、书单与资源（深度补充用，非学习前提）

**必读**：《JavaScript 高级程序设计（第4版）》核心章节｜《你不知道的JavaScript（上卷）》｜《Effective TypeScript（第2版）》｜《Vue.js 设计与实现》（进阶原理）｜《CSS 新世界》。
**官方文档**（最高频读物）：cn.vuejs.org（3.5）> TS Handbook > Pinia / Vue Router / Vite / VueUse。
**明确不学**：jQuery、Vue 2 / Vuex / Options API 深研、Webpack 配置、Bootstrap。

---

## 十、综合面试问答与优秀解答（中高级标准）

**Q1：Vue 3 响应式原理，从数据修改到视图更新的完整链路？**

> ① 组件渲染函数被包装为 `ReactiveEffect` 运行；② 模板读取数据触发 Proxy 的 `get` → `track` 将当前 effect 记入该属性的依赖集合（存储在 `targetMap: WeakMap<target, Map<key, Set<effect>>>`）；③ 修改数据触发 `set`，值变化时 `trigger` 通知依赖集合；④ effect 并非立即执行，而是入队，在 `nextTick` 的微任务中**去重后批量执行**；⑤ 重新运行渲染函数生成新虚拟 DOM，经带 Patch Flag 优化的 diff 得到最小变更集，应用到真实 DOM。

**Q2：Vue 2 与 Vue 3 响应式差异？**

> Vue 2 用 `Object.defineProperty`：初始化递归遍历整棵数据树（大对象初始化慢）、无法监听属性增删（需 `$set`）和数组索引/length（需重写七个数组方法）；Vue 3 用 `Proxy`：可拦截一切读取/修改、**惰性深层代理**（访问到才递归）、数组与增删天然支持，配合 WeakMap 存依赖避免内存泄漏。局限：Proxy 无法代理基本类型（所以有 ref）。

**Q3：nextTick 的作用与原理？为什么要存在？**

> 修改响应式数据后 DOM 是**异步批量更新**的（一轮多次修改只渲染一次，性能优化）。`nextTick(cb)` 返回 Promise，将回调注册到“更新队列刷新之后”的微任务，cb 中能读到最新 DOM。等价写法 `await nextTick()`。

**Q4：Vue 组件间通信所有方式及选型？** → 背 5.6 选型表，附一句原则：“优先 props/emits 单向流，跨层用 provide，全局用 Pinia，拿子组件方法用 defineExpose + ref”。

**Q5：v-model 的本质？自定义组件如何实现？**

> `v-model="x"` 是 `:modelValue="x"` + `@update:modelValue="v => x = v"` 的编译期语法糖。Vue 3.4+ 组件内用 `defineModel()` 宏一行实现，支持多 `v-model:title` 与修饰符处理。原生 input 上则是 `:value` + `@input`。

**Q6：computed 缓存原理？**

> computed 是一个带 `dirty` 标记的惰性 effect：首次读取才求值；依赖变化时只把 `dirty` 置 true 并通知读取过它的 effect，**并不立即重算**；下次读取发现 dirty 才重新求值并更新缓存。因此“依赖不变时多次读取零开销”。

**Q7：KeepAlive 原理与注意点？**

> 内部维护一个缓存 Map（组件 key → vnode）+ LRU 淘汰策略；卸载时不销毁而是失活，激活时从缓存取回（触发 `onActivated/onDeactivated` 而非重新挂载）。注意：`include` 按**组件 name** 匹配（`<script setup>` 需 `defineOptions({ name })`）；被缓存页面的数据请求刷新应放 `onActivated` 而非 `onMounted`（后者只执行一次）。

**Q8：事件循环输出题（见 3.3.10）答题框架？**

> “先说模型：同步栈清空 → 清空全部微任务 → 一个宏任务 → 循环；再标 Promise.then/await 后续代码是微任务、setTimeout 是宏任务；最后逐行标注执行顺序。”（附 `await` 细节：await 右侧表达式同步执行，其后的代码进入微任务。）

**Q9：跨域是什么？有哪些方案？你们项目怎么解决的？**

> 同源策略限制 JS 读取非同源响应。标准方案 CORS（区分简单请求/预检 OPTIONS，携带 Cookie 需 withCredentials + 精确 Allow-Origin）；开发期 Vite proxy；生产期 Nginx 反向代理实现同域部署（我们项目的做法，一劳永逸）；历史方案 JSONP（利用 script 标签不受限，仅 GET）。

**Q10：强缓存与协商缓存？项目中如何配置？**

> 见 7.4.4。答题必带实战：“hash 资源 `max-age=31536000, immutable` 永久缓存；`index.html` `no-cache` 每次协商；解决‘发版后用户看到旧版本’的问题”。

**Q11：Vite 为什么快？**

> 开发期：①不打包，浏览器原生 ESM 按请求编译，启动时间与项目规模解耦；②依赖预构建用 esbuild（Go 编写，比 JS 工具链快 10~100 倍）将 CommonJS 依赖转为 ESM 并合并请求；③ HMR 基于模块图精确失效，只更新受影响模块。生产期仍用 Rollup 打包（成熟的 Tree Shaking 与分包）。

**Q12：首屏优化完整方案（开放题答题模板）**

> 按“度量→网络→资源→渲染”四步答：先 Lighthouse 定位瓶颈；网络层 CDN + Gzip/Brotli + HTTP2 + 关键资源 preload；资源层路由懒加载 + 按需引入 + 分包 + 图片优化；渲染层 SSR/骨架屏 + 减小主包 + 首屏数据并行请求；最后给出量化结果（如 LCP 3.2s → 1.4s）。

**Q13：手写题清单（全部达到默写级别）**：防抖/节流（3.4 Q1）｜深拷贝带循环引用（见下）｜Promise 并发限制（见下）｜迷你 reactive（7.1）｜`MyPartial/MyPick`（4.5）。

```typescript
// 深拷贝（WeakMap 破循环引用 + 特殊对象处理）
function deepClone<T>(obj: T, map = new WeakMap()): T {
  if (obj === null || typeof obj !== 'object') return obj          // 基本类型/函数直接返回
  if (obj instanceof Date) return new Date(obj) as T
  if (obj instanceof RegExp) return new RegExp(obj) as T
  if (map.has(obj)) return map.get(obj)                             // ★ 循环引用直接返回缓存
  const clone = (Array.isArray(obj) ? [] : {}) as T
  map.set(obj, clone)                                               // 先存引用再递归
  ;(Reflect.ownKeys(obj) as (keyof T)[]).forEach(key => {
    clone[key] = deepClone(obj[key], map)
  })
  return clone
}

// Promise 并发限制调度器（面试高频）
async function asyncPool<T>(limit: number, tasks: (() => Promise<T>)[]): Promise<T[]> {
  const results: T[] = new Array(tasks.length)
  const executing = new Set<Promise<void>>()
  for (let i = 0; i < tasks.length; i++) {
    const p = tasks[i]().then(res => { results[i] = res; executing.delete(p) })
    executing.add(p)
    if (executing.size >= limit) await Promise.race(executing)   // 池满则等最快的完成腾位
  }
  await Promise.all(executing)   // 等剩余的收尾
  return results
}
// 用法：100 个接口、最多同时 5 个 —— asyncPool(5, urls.map(u => () => fetch(u)))
```

**Q14：移动端转型者的差异化叙事（非技术题但决定成败）**

> 面试话术模板：“我做移动端 X 年，架构与性能优化的方法论是通用的——RecyclerView 复用思想帮我快速理解虚拟列表；原生渲染管线经验让我做重排重绘优化时有直觉；混合开发经验让我和原生团队的协作成本为零。前端对我来说是能力半径的扩展而不是从零开始。”

---

## 十一、最终检验标准（达成即中高级）

- [ ] 闭包/this/事件循环/Promise 能给别人讲明白（费曼检验）
- [ ] 默写：防抖节流、深拷贝、并发调度、迷你 reactive、MyPartial
- [ ] 讲清：Proxy 响应式完整链路、diff 与 key、nextTick、computed 缓存
- [ ] 讲清：URL→渲染全流程、强缓存/协商缓存、CORS 预检、JWT 双 Token
- [ ] 独立完成并部署项目三全部功能清单，Lighthouse ≥ 85
- [ ] 拿到新需求能自主完成“组件拆分 → 状态设计（组件内 or Pinia）→ 接口层 → 权限”的架构决策
