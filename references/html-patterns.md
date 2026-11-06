# PSD 源页面写法与反例

配合 [SKILL.md](../SKILL.md) 使用。所有例子都按“会被 webpage-to-psd 读取”这个前提写，而不是单纯追求浏览器效果。

可直接打开的完整骨架见 [template.html](template.html)（`<main>` 分组、整数几何、绝对定位浮条、`min-width`、纯色面都已就位）。

## 一、一个模块的推荐写法

```html
<section class="product-section" id="cryogenic">
  <div class="wrap">
    <p class="eyebrow">PRODUCT SYSTEMS</p>
    <h2 class="section-title">新连通产品系统</h2>
    <p class="section-sub">从应用场景出发，发现适合您的装备组合</p>

    <div class="product-grid">
      <a class="product-card" href="/products/hose">
        <img class="product-card-image" src="assets/hose.png" width="320" height="180"
             alt="真空绝热软管">
        <h3 class="product-card-title">真空绝热软管</h3>
        <span class="product-card-arrow">→</span>
      </a>
    </div>
  </div>
</section>
```

对应 CSS：

```css
.product-section { padding: 80px 0; background: #fff; }
.product-card {
  background: #f4f6f8;          /* 纯色 → 原生形状 */
  border: 1px solid #e7edf2;    /* 边框写在同一个元素上 */
  border-radius: 0;             /* 四角可独立设置 */
  padding: 12px 14px 18px;
}
.product-card-image { display: block; width: 100%; height: 165px; object-fit: contain; }
```

这一张卡片只产生：1 个形状原子（背景 + 边框）+ 1 个图片原子 + 3 个文字行。

## 二、同一张卡片的反例

```html
<!-- 不要这样 -->
<div class="card">
  <div class="card-bg"></div>
  <div class="card-border"></div>
  <div class="card-shadow"></div>
  <div class="card-media">
    <span class="iconfont">&#xe601;</span>
    <img src="data:image/png;base64,iVBORw0..." alt="">
  </div>
  <div class="card-title"><span>真</span><span>空</span><span>软</span><span>管</span></div>
</div>
<style>
  .card { position: relative; transform: translate(-50%, -50%); }
  .card-border { position: absolute; inset: 0; border: 1px solid #e7edf2; }
  .card-bg { position: absolute; inset: 0; background: linear-gradient(#fff, #f4f6f8), url(bg.png); }
  .card::before { content: ""; position: absolute; left: 0; top: 0; width: 4px; height: 100%; background: #003575; }
</style>
```

问题与代价：

| 写法 | 后果 |
|---|---|
| 三层 div 叠底色/边框/阴影 | 3 个形状原子，Photoshop 里 3 个图层 |
| 逐字 `<span>` | 每个字都要测边界，文字行被拆碎 |
| 图标字体 | 图标变成文字层，字体缺失时显示错误 |
| `data:` 图片 | DOM 与素材膨胀，图层名丢失 |
| `linear-gradient + url()` 多背景 | prepare 报错，需要图片回退 |
| `translate(-50%,-50%)` | 形状按外接矩形绘制，位置漂移 |
| `::before` 画竖条 | 多一个 overlay 原子和整份伪元素样式 |

## 三、CSS 黑白名单

**可以放心用**

- 盒模型：`width/height/margin/padding/box-sizing`
- 布局：`display:flex/grid`、`gap`、`align-*`、`justify-*`
- 绘制：`background-color`、`border`、`border-radius`、单层 `linear-gradient`、`box-shadow`（受支持 profile）
- 文本：`font-family/size/weight/style`、`line-height`、`letter-spacing`、`color`、`text-align`、`text-transform`
- 图片：`object-fit: fill/contain/cover/none/scale-down`、`object-position`
- 只有一个平移的 `transform: translate(x, y)`

**不要用（会失败、回退或丢效果）**

- `filter`、`backdrop-filter`、`mix-blend-mode`
- `clip-path`、`mask`、`mask-image`
- `transform: rotate/scale/skew/matrix` 的缩放旋转部分
- 多重 `background-image`、`background-blend-mode`
- `outline`、`border-image`（抓取不读取）
- 仅靠 `text-decoration` 画可见下划线（用 `border-bottom` 或独立元素）
- `writing-mode: vertical-*`、`text-shadow`
- `content:"正文"` 承载文案

## 四、图片与图标

**推荐**

```html
<img class="avatar" src="assets/avatar.jpg" width="96" height="96" alt="张先生">
```

```css
.avatar { width: 96px; height: 96px; border-radius: 50%; object-fit: cover; }
```

**内联 SVG 图标**

```html
<a class="contact-link" href="tel:02885790225">
  <svg class="icon" viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
    <path d="M6.6 2.5 3.8 4.1c-2.9 4.8 4.7 15 10.5 17.4 2.8 1.2 5.6-.8 6.2-2.6l-4.1-3.5-2.3 2.1c-3.5-1.8-6.2-4.8-7.3-8.1l2.6-1.9z"/>
  </svg>
  <span>028-85790225</span>
</a>
```

每个图标一个自包含 `<svg>`；不要 `<use href="sprite.svg#icon">`、不要图标字体、不要把图标做成背景图。

## 五、文本与排版

**推荐**

```html
<p class="intro">从真空绝热结构到低温制冷，支持超导系统的专业装备。</p>
```

**避免**

```html
<p class="intro"><span>从</span><span>真</span><span>空</span>……</p>
```

- 一个段落一个文本节点；刻意换行用 `<br>`。
- 行高用整数 px（如 `28px`），不要 `line-height: 1.65` 让浏览器算出小数再被四舍五入。
- 不要用 `letter-spacing` 拉开字距做版式；需要字距时保持整段一致。
- 表单默认值、按钮文案、占位提示都要是可见文本节点。

## 六、轮播、标签页、弹窗

**轮播**：非活动页直接隐藏，不要横向移出画布。

```html
<div class="slides">
  <figure class="slide is-active"><img src="assets/a.jpg" alt="低温传输管线"></figure>
  <figure class="slide" hidden><img src="assets/b.jpg" alt="低温阀箱"></figure>
</div>
```

```css
.slide[hidden] { display: none; }
```

**标签页**：只渲染当前面板，其余 `hidden`；不要靠 `opacity:0` 或移出视口。

**弹窗**：默认关闭（`<dialog>` 不加 `open`，或 `display:none`）；PSD 只表达默认关闭态。

**固定浮层**：全页最多一个，DOM 放最后，宽度固定；不要用 fixed 做页头或主要内容。

## 七、命名与分组

图层名来源顺序是 `文字 → alt → class → tag`，所以：

| class | 生成的图层名 | 评价 |
|---|---|---|
| `product-card` | product-card | 可读 |
| `hero-title` | hero-title | 可读 |
| `btn primary large` | btn primary large | 偏长但可读 |
| `a` / `btn` / `i` | a / btn / i | 不可读 |

- 每个模块第一个 class 用语义名；Tailwind 项目可在工具类前加一个语义 class：`class="product-card flex items-center"`。
- 模块是 `<main>` 的直接子元素（通常写成 `<section>`），组名取组内第一个 `h1/h2/h3`；页头/页脚/浮层留在 `<main>` 外面。没有 `<main>` 时整页会并成一组、组名取第一个 `h1`。
- 图片 `alt` 会进图层名，写“低温真空阀门”而不是“image1”。

## 八、框架与构建产物

- **React/Vue**：交给转换器的是构建后的静态 HTML；确保 hydration 不改结构、不改可见样式。可见内容不要放在 portal/teleport 里。
- **Tailwind**：实用类可以保留，但给每个视觉盒子加语义 class，避免图层名变成 `flex items-center gap-4`。
- **CSS-in-JS**：运行时注入样式可以，但类名常是哈希；给块级元素补语义 class。
- **图标库**：构建时内联成 `<svg>`，不要运行时注入 sprite 或字体图标。
- **SSG/SSR**：生成后不要再由客户端脚本替换文本或图片。

## 九、症状对照表

| 症状 | 常见原因 | 修法 |
|---|---|---|
| 转换慢、图层特别多 | wrapper 多、伪元素多、每张卡多层叠 | 合并盒子、删 wrapper、伪元素 ≤20 |
| 图片被栅格或 prepare 报错 | 图片或祖先有 transform/filter/clip-path | 去掉效果，用布局或圆角实现 |
| 文字层错位/缺行 | 逐字 span、竖排、text-shadow | 一段一节点、横排、去阴影 |
| 抓取不稳定 | 自动轮播、定时器、滚动动画 | 静态首帧、停掉动画与定时器 |
| 图层名是 a/btn/i | class 无语义 | 加语义 class、写 alt |
| 页面两侧有白边、整页像浮在中间的一块 | 根容器或外层 div 写了 `width`/`max-width`，或给区块套了定宽 | 根容器宽度交给视口，只给 `.wrap` 定宽；区块背景横向到边 |
| PSD 里没有边框/下划线 | 用了 outline/text-decoration | 改 border |
| 页面底部被截断 | 高度超过 30000px | 拆页或压缩内容 |
| 固定浮层跑到页面中间 | 用了超高视口抓取 | 保持默认 1080 高视口，让转换器处理 fixed |

## 十、验收

生成完成后按 `webpage-to-psd` 的 `capture` → `prepare` 跑一遍，确认 `stable: true`、`atoms` ≤ 400、`pages` ≤ 8、`imageFallbacks` 为空。数值达标再进入 build/native，不要等 Photoshop 阶段才发现图层爆炸。
