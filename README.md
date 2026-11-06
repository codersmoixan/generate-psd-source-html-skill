# generate-psd-source-html 使用说明

生成“将要交给 PSD 转换器”的静态 HTML 页面。它不负责转 PSD，而是约束页面的结构、绘制方式、文字、图片和动效，让 [webpage-to-psd](../webpage-to-psd/SKILL.md) 能快速读取、少产生图层、保持文字与形状可编辑。

## 解决什么问题

- 转换慢、图集页多、Photoshop 重建久
- 卡片/按钮/边框被拆成多层，图层数量爆炸
- 图片被栅格化，或 prepare 直接报错
- 文字层错位、缺行，图标变成文字
- 抓取不稳定，前后 DOM 不一致

核心判断标准不是“浏览器里好不好看”，而是**绘制原子少、文字是文本、图片有原文件、状态静态**。

## 在 DSH 中使用

直接描述“生成一个之后要转 PSD 的页面”，技能会自动匹配；也可以显式调用：

```text
/generate-psd-source-html 生成一个产品中心页面，1920 宽，之后要转 PSD
```

```text
/generate-psd-source-html 检查 D:/站点/preview 这个页面适不适合转 PSD，
哪里会导致图层过多或图片回退
```

```text
/generate-psd-source-html 把这个页面的卡片结构改成 PSD 友好的写法，
不要改变视觉稿
```

适用：从零生成 PSD 源页面、改造现有页面、评审页面是否能顺利转换。
不适用：直接生成 PSD（用 `webpage-to-psd`）、纯截图转 PSD、需要交互态/多断点但未明确说明的页面。

## 规则速览

### 预算

| 指标 | 建议上限 |
|---|---|
| 可见元素 | 800 |
| 文本字符 | 20000 |
| 绘制原子 | 400 |
| 文字行（≈文字层） | 200 |
| 图集页 | 8 |
| 伪元素 | 20 |
| 字体家族 / 字重 | 3 / 3 |
| 图片总量 | 30MB |
| 页面高度 | 12000px（硬限 30000） |
| 固定浮层 | 1 |

### 八条硬性约束

1. 图片及祖先只用纯平移 `transform`，禁止 `rotate/scale/skew/filter/clip-path/mix-blend-mode`。
2. 背景最多一层 `linear-gradient` + 一张图，禁止多重背景。
3. 可见边框用 `border`，不要用 `outline`/`border-image`/`text-decoration`。
4. 正文不要写在 `content:"..."` 里。
5. 原图必须是本地可访问文件。
6. 页面高度不超过 30000px。
7. 不要自动轮播、定时器改 DOM、延迟渲染。
8. 阴影只用受支持 profile，多层或模糊内阴影会失败。

### 写法要点

- 一个视觉盒子一个元素，底色/边框/圆角/阴影写在一起
- 纯色用 `background-color`，圆角用 `border-radius`
- 图标用自包含内联 `<svg>`，不要图标字体
- 一个段落一个文本节点，禁止逐字 `<span>`
- 图片用 `<img>` + 本地 `src` + `width/height` + `alt`
- 轮播非活动页用 `display:none`，不要移出画布
- 语义 class 会变成图层名，`product-card` 远好于 `a`

## 与 webpage-to-psd 的分工

| 技能 | 职责 |
|---|---|
| `generate-psd-source-html` | 约束 HTML/CSS 的产出，控制元素、原子、文字层、图片和稳定性 |
| `webpage-to-psd` | 抓取、分层、图集、Photoshop 原生重建、交付闸门 |

生成完成后按 `webpage-to-psd` 的 `capture` → `prepare` 实测：`stable: true`、`atoms ≤ 400`、`pages ≤ 8`、`imageFallbacks` 为空，再进入 build/native。

## 文件说明

| 文件 | 内容 |
|---|---|
| `SKILL.md` | 执行规则：预算、硬性约束、结构/文字/图片/状态/命名、自检与交接 |
| `references/html-patterns.md` | 正反例、CSS 黑白名单、图标签名与轮播写法、框架注意事项、症状对照表 |
| `README.md` | 本文件，面向使用者 |

执行以 `SKILL.md` 和 `references/html-patterns.md` 为准；README 只做入口介绍。

## 边界

- 不生成 PSD，不修改 `webpage-to-psd` 的脚本。
- 不为了转换而改变品牌、文案或视觉稿；只调整实现方式。
- 确实必须使用不支持的效果时，在交付说明里标注，由 PSD 阶段决定是否启用栅格回退，不能偷偷换成整页截图。
