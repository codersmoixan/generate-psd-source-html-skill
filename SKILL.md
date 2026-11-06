---
name: generate-psd-source-html
description: Author or review static HTML/CSS that will be converted to PSD by webpage-to-psd. Use when generating a page destined for PSD export, when conversion is slow or produces raster fallbacks, or when constraining DOM size, paint atoms, text, images, fonts and animations so layers stay editable and the pipeline stays fast. Do not use to perform the PSD conversion itself.
---

# 生成 PSD 源页面 HTML

本技能约束“要交给 webpage-to-psd 的静态页面”怎么生成。目标不是浏览器里好看，而是让转换器**少产生绘制原子、文字保持可编辑、图片可追溯、抓取状态稳定**。转 PSD 本身交给 `webpage-to-psd`，这里只负责源页面。

## 先定三件事

- 视口宽度：用户给了就用给的，没给按 1920 设计；一个页面只做一个静态状态。
- 页面高度：建议 ≤ 12000px，硬上限 30000px。
- 必须保留的效果：先确认哪些渐变、阴影、浮层非有不可，再决定是否接受栅格回退。

## 一、预算（生成前先定，生成后实测）

| 指标 | 建议上限 | 说明 |
|---|---|---|
| 可见元素 | 800 | 每个元素约 2KB DOM 数据，且逐层计算样式 |
| 文本字符 | 20000 | 抓取逐字测量边界，字符越多越慢 |
| 绘制原子 | 400 | 背景/边框/阴影/文字行/图片/图标各算一个 |
| 文字行（≈文字层） | 200 | 直接决定 Photoshop 重建时间 |
| 图集页 | 8 | 每页还要渲染白/黑两张 |
| 伪元素 | 20 | 每个都触发整份计算样式导出 |
| 字体家族 / 字重 | 3 / 3 | 影响 Photoshop 字体匹配 |
| 图片总量 | 30MB | 原图直接进 PSD |
| 固定浮层 | 1 | fixed 元素按滚动 0 处入图 |

超预算时优先删装饰层、合并盒子、减少伪元素；不要删正文。

## 二、硬性约束（不满足会报错或明显失真）

1. 图片及其祖先只用纯平移 `transform`；禁止 `rotate/scale/skew`、`filter`、`clip-path`、`mix-blend-mode`，否则 prepare 直接拒绝。
2. 背景最多“一层 linear-gradient + 一张图”；禁止多重背景、`background-blend-mode`。
3. 可见边框用 `border`，不要用 `outline`、`border-image`；下划线不要依赖 `text-decoration`——这些属性不在抓取范围，PSD 里会丢。
4. 正文不要写在 `content:"..."` 里：伪元素只还原背景与圆角，文字会丢。
5. 原图必须是本地可访问文件；缺图即失败，不能用截图顶替。
6. 页面高度硬上限 30000px；超过直接拒绝。
7. 首屏不要自动轮播、定时器改 DOM、延迟渲染；截图前后 DOM 必须一致，否则抓取失败。
8. 阴影只用受支持 profile：零偏移零模糊的外描边/内描边，或 `webpage-to-psd` 列出的柔阴影；多层阴影、模糊内阴影会让 native 阶段失败。

## 三、结构与绘制

- **一个视觉盒子一个元素**：底色、边框、圆角、阴影写在同一元素上，不要拆成多层 div 叠出来。
- 纯色面用 `background-color`；渐变/图片背景只能进图集，纯色才能成为原生形状层。
- 圆角用 `border-radius`（四角可不同），不要用 `clip-path` 或遮罩图。
- 装饰尽量不用 `::before/::after`；必须用时只做纯色块，且全页 ≤ 20 个。
- 定位用 flex/grid，不要用 `translate(-50%,-50%)` 居中或 `scale` 做自适应。
- 图标用自包含内联 `<svg>`（每个图标一个），不要图标字体、不要外部 sprite/`<use>`。
- 分隔线用容器的 `border`，不要为每条线加一个空 div。
- 删掉纯定位 wrapper，DOM 层级建议 ≤ 8。

## 四、文字

- 一个段落一个文本节点；**禁止逐字/逐词 `<span>`**（会按字符测边界并增加换行图层）。
- 文字必须是真实文本：不要图片字、SVG 文本、canvas 文字、图标字体转文字。
- 行高、字距用整数 px；避免 `text-shadow`、极端 `letter-spacing`、竖排 `writing-mode`。
- 字体用本地 woff2，家族与字重收敛；确保 Photoshop 端能装到同名字体。
- 表单默认值要渲染成可见文本，不要只靠 `value` 属性。
- 一个文案不要为了样式拆成多个文本节点；用 `<br>` 或独立段落表达刻意换行。

## 五、图片

- 用 `<img>` + 本地 `src` + 明确 `width`/`height` + 可读 `alt`。
- 不要 base64 data URI；不要多候选 `srcset`（1920 下必须唯一）。
- `object-fit` 只用 `fill / contain / cover / none / scale-down`。
- 图片尺寸接近展示尺寸即可，避免 6000px 原图拖慢构建与保存。
- 背景图只允许单张；`background-size/position/repeat` 用常见值。
- 圆形头像用正方形 + `border-radius:50%`；不要对长方形做椭圆圆角。

## 六、状态与动效

- 一个页面只做一个静态状态；轮播、标签页、弹窗不要自动切换。
- 非活动轮播页用 `display:none`/`visibility:hidden`，不要用 `translateX` 移出画布。
- 不要滚动触发 reveal、入场动画、`setInterval` 改样式；抓取会滚整页并比对前后 DOM。
- 需要 JS 渲染的内容，确保在 `load` 前完成且之后不再变。
- 固定/悬浮控件最多一个，放在 DOM 末尾；不要用 fixed 承载主要内容。

## 七、命名与分组

- 模块用 `<section>`/`<header>`/`<footer>` + 可见标题，转换器按标题生成图层组。
- 第一个语义 class 要可读：图层名取 `文字 / alt / class`，`class="product-card"` 远好于 `class="a"`。
- 图片 `alt` 写清楚，它会成为图层名。
- 保持 DOM 顺序 = 视觉顺序；少用负 `z-index` 重排。

## 八、生成后自检

### 静态扫描（启发式）

在源文件里搜索并清零：

```text
transform: rotate|scale|skew
filter:|backdrop-filter:|clip-path:|mix-blend-mode:
background-image:  (出现多个 url/gradient)
outline:|border-image:
data:image/
::before / ::after  (统计数量)
position: fixed      (统计数量)
```

### 实测（以转换器输出为准）

用 `webpage-to-psd` 跑一次 `capture` + `prepare`，看：

- 抓取：`stable: true`、无 warnings、无 `assetFailures`
- `prepare`：`atoms` ≤ 400、`pages` ≤ 8、`imageFallbacks` 为空
- `web_dom.json`：`elements` ≤ 800，文本字符量在预算内

超预算就回去合并盒子、删装饰、拆长文；不要靠提高渲染倍率或换机器解决。

## 九、交接

- 页面完成后交给 `webpage-to-psd`，按它的阶段命令抓取、构建、原生重建与闸门。
- 确实必须使用不支持的效果时，在交付说明里标注，由 PSD 阶段决定是否启用 `--allow-image-raster-fallback`；不要偷偷改用整页截图。
- 本技能不生成 PSD，也不修改转换器脚本。

## 十、按需阅读

| 什么时候读 | 读哪篇 |
|---|---|
| 需要完整正反例、CSS 黑白名单、框架写法 | [references/html-patterns.md](references/html-patterns.md) |
