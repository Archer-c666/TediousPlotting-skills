# 设计系统

本文档定义本 skill 所有输出共享的视觉语言，是整个 skill 的**唯一真源**与**核心教材**。**每一条规则都基于感知科学或排印传统**——理解背后的逻辑比记住具体数值更重要。

**怎么用它：** 把这套语言**内化**，然后应用到你构建的任何可视化上——无论它是 11 种标准习语之一，还是为数据适配出的变体、组合出的多编码视图，乃至全新的图类型。`assets/` 里的模板只是这套语言落到某种习语上的**示范**；当你调整或自创时，请保留每条规则的**意图**（为什么这样配色、为什么这样排版、为什么这样动），而不是机械照抄某个模板的字面写法。原则恒定，习语可变。

---

## 目录

1. [配色系统](#1-配色系统)
2. [字体排印](#2-字体排印)
3. [布局与构图](#3-布局与构图)
4. [交互模式](#4-交互模式)
5. [动效](#5-动效)
6. [焦点态与键盘导航](#6-焦点态与键盘导航)
7. [空状态与错误](#7-空状态与错误)
8. [背景氛围](#8-背景氛围)
9. [响应式](#9-响应式)
10. [无障碍核查清单](#10-无障碍核查清单)

---

## 1. 配色系统

### 1.1 基础色板（CSS 变量）

所有模板必须使用以下变量。仅当用户明确要求换主题时才覆盖。

```css
:root{
  /* 表面层级 —— 三档背景营造空间纵深感 */
  --bg-0:      #0b0d12;   /* 页面背景，最底层 */
  --bg-1:      #11141b;   /* 面板/卡片背景 */
  --bg-2:      #181c25;   /* 悬浮/提升表面（tooltip、按钮 hover） */

  /* 分隔线 —— 用半透明白而非纯灰，跨主题切换时不突兀 */
  --line:      rgba(255,255,255,0.08);   /* 默认分隔 */
  --line-2:    rgba(255,255,255,0.16);   /* 强调边框（聚焦、tooltip 边） */

  /* 文字层级 —— 三档明度建立信息优先级 */
  --ink:       #ecedef;   /* 主文本，95% 白 */
  --ink-dim:   #9aa0aa;   /* 次文本，副标题、说明 */
  --ink-mute:  #5b6068;   /* 三级文本，标签、占位 */

  /* 强调色 —— 稀缺为贵 */
  --accent:    #f0c674;   /* 暖琥珀，主强调（聚焦、激活、CTA） */
  --accent-2:  #e36a4a;   /* 赤陶，次强调（对比/危险/告警） */

  /* 语义色 —— 仅在需要时启用 */
  --good:      #7fb8a4;   /* 鼠尾草绿（成功/正面） */
  --warn:      #d6b85a;   /* 麦秆黄（警示） */
  --bad:       #e36a4a;   /* 赤陶（错误/负面） */
}
```

> 表面三档（bg-0/1/2）的明度差约 4%。在 OLED 上能拉开空间感，又不至于在 LCD 上显成条带。

### 1.2 分类色板

7 种色相，明度统一控制在 60-70 区间，遵循以下挑选原则：

- 红绿不单独承载语义（保护 8% 男性的红绿色弱）
- 转灰度后通过明度差仍可区分
- 同时适配冷暖数据主题

```javascript
const categoricalPalette = [
  "#e36a4a",  // 赤陶（terracotta）
  "#f0c674",  // 琥珀（amber）
  "#7fb8a4",  // 鼠尾草绿（sage）
  "#c98a5b",  // 焦糖（caramel）
  "#b88ec2",  // 丁香（lilac）
  "#6a9bd1",  // 钢蓝（steel）
  "#d6b85a",  // 麦秆（straw）
];
```

**重要：** 分类色相不要超过 7-8 种。色相互相干扰会让区分变得困难（Healey & Enns 2012 关于预注意特征的研究）。再多就把小类合并成"其他"。

### 1.3 顺序色板

编码**有序数据**时使用感知均匀的单色相梯度，优先用保留色相、仅变明度/饱和度的插值器：

```javascript
// 好：保留色相，变明度（暗→暖琥珀）
const sequentialAmber  = d3.interpolateRgb("#3a2a1a", "#f0c674");
const sequentialOcean  = d3.interpolateRgb("#162028", "#6a9bd1");
const sequentialEarth  = d3.interpolateRgb("#1f1810", "#c98a5b");

// 或用 d3-scale-chromatic 的感知均匀方案
d3.interpolateYlOrRd    // 顺序暖色
d3.interpolateViridis   // 感知均匀，色盲安全
d3.interpolateCividis   // 同上，且印刷友好
```

### 1.4 发散色板

围绕**中性中点**编码 +/-，双色经中性灰渐变。**绝对禁止**从纯红渐变到纯绿。

```javascript
const diverging = d3.scaleDiverging()
  .interpolator(d3.interpolateRdBu)   // 红→白→蓝（色盲安全）
  .domain([-1, 0, 1]);

// 或自定义暖冷对：
const customDiverging = d3.scaleDiverging()
  .interpolator(d3.piecewise(d3.interpolateRgb, ["#6a9bd1", "#cdd1d6", "#e36a4a"]))
  .domain([min, 0, max]);
```

### 1.5 配色十条（含感知学依据）

1. **不要单独依赖颜色。** 色相必须搭配第二通道（标签、形状、明度）。冗余编码是无障碍底线，不可妥协。
2. **色相只编码定类（categorical）。** 有序数据应该用明度或饱和度——人眼无法感知"黄 > 蓝"这种色相顺序。
3. **分类色相 ≤ 7-8 种。** 超过则色相干扰严重，预注意性能崩盘。
4. **背景与前景对比 ≥ 4.5:1**（WCAG AA）。对所有标记填色与 `--bg-0` 做对比测试。
5. **强调色稀缺为贵。** `--accent` 仅用于：焦点元素、悬停态、激活选区、主要 CTA。撒得到处都是，"弹出效应"就死了（preattentive popout 需要稀缺性）。
6. **暖色优先放在前景。** 暖色（红橙黄）在视网膜上的色差敏感度更高，自然向前；冷色（蓝绿紫）后退。色彩透视。
7. **饱和度做层级。** 高饱和元素在前，低饱和元素退后——比仅用明度更柔和的层级方式。
8. **避免纯白纯黑。** `#ffffff` 在深色背景上炫光；`#000000` 在浅色背景上像黑洞。用 `--ink (#ecedef)` 和 `--bg-0 (#0b0d12)`。
9. **同一色相的不同明度可表达同族类目。** 父类用饱和琥珀，子类用稍亮的琥珀变体——保持视觉血缘关系。
10. **测试灰度模式。** `filter: grayscale(100%)` 套上去看图表是否仍可读。能读 = 信息没有单靠色相承载。

---

## 2. 字体排印

### 2.1 字体堆栈

模板必须包含此 `<link>`：

```html
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,300;9..144,400;9..144,600;9..144,800&family=JetBrains+Mono:wght@300;400;500&family=Inter+Tight:wght@300;400;500;600&display=swap" rel="stylesheet">
```

| 角色                         | 字体                     | 选用理由                                         |
| ---------------------------- | ------------------------ | ------------------------------------------------ |
| **大标题 / Hero**      | `Fraunces`（可变衬线） | 独特的光学尺寸轴；衬线建立视觉层级；编辑设计感强 |
| **正文 / UI**          | `Inter Tight`          | x-height 高、字距紧凑、小字号下可读性极佳        |
| **数字 / 标签 / 等宽** | `JetBrains Mono`       | 表格化数字对齐；带出"仪表盘/技术"声调            |

**禁用字体：** Inter（普通版）、Arial、Roboto、Helvetica、system-ui、Times New Roman。这些是 AI 默认字体味，立刻让作品掉一个档次。

### 2.2 类型层级

```css
/* Hero 标题 —— Fraunces 细体，斜体高亮单词 */
h1 {
  font-family: 'Fraunces', serif;
  font-weight: 300;
  font-size: clamp(38px, 4.4vw, 64px);
  line-height: 1.02;
  letter-spacing: -.02em;
}
h1 em {
  font-style: italic;
  font-weight: 400;
  color: var(--accent);
}

/* 二级标题 */
h2 {
  font-family: 'Fraunces', serif;
  font-weight: 300;
  font-size: 26px;
  letter-spacing: -.01em;
}

/* 引导文 */
.lede {
  font-size: 14px;
  line-height: 1.55;
  color: var(--ink-dim);
  max-width: 540px;
}

/* 小标签 / 类目 —— 等宽 + 大写 + 宽字距 */
.eyebrow {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  letter-spacing: .28em;
  text-transform: uppercase;
  color: var(--ink-mute);
}

.label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  letter-spacing: .22em;
  text-transform: uppercase;
}

/* 数值展示 —— Fraunces 大数 + JetBrains Mono 单位 */
.value {
  font-family: 'Fraunces', serif;
  font-weight: 300;
  font-size: 22px;
  font-variant-numeric: tabular-nums lining-nums;
}
.value small {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: var(--ink-mute);
  margin-left: 4px;
}
```

### 2.3 字体规则六条

1. **恰好混搭 3 种字体家族。** 再多视觉嘈杂，再少层级扁平。
2. **`<em>` 仅用于英雄词。** 整页只有一个高亮焦点，绝不多处。
3. **所有数值用表格化数字。** JetBrains Mono 本身等宽；比例字体加 `font-variant-numeric: tabular-nums`。这是数据展示的最低门槛。
4. **标签用等宽 + 大写 + 宽字距 (`.22em-.28em`)。** 这种模式呼应科学/编辑类 UI（参见 Pentagram、Bloomberg 的设计语言）。
5. **行高 1.02 给标题，1.55 给正文。** 短句子可以挤、长段落必须放。
6. **字号选 clamp() 而非媒体查询。** 大屏稳，移动端不至于太大。

---

## 3. 布局与构图

### 3.1 页面外框

```css
.frame {
  position: relative;
  z-index: 1;
  max-width: 1400px;
  margin: 0 auto;
  padding: 48px 56px 64px;
}
```

### 3.2 头部区（非对称布局）

`grid 1fr auto`：左边一根独立标题柱，右边一块紧凑的元信息块。这种非对称比传统居中标题更有编辑感（参考杂志版面）。

```html
<header class="title">
  <div>
    <div class="eyebrow"><span class="dot"></span> [上下文] · [图类型] · [主题]</div>
    <h1>[主标题左] <em>[斜体高亮]</em><br>[主标题右]</h1>
    <p class="lede">[1-2 句引导，结尾给一个交互提示]</p>
  </div>
  <div class="meta">
    <b>[项目]</b>   /   [学科] <br>
    Dataset · [来源 / 年份] <br>
    Idiom · <b>[图名]</b>   /   Marks · [主标记]
  </div>
</header>
```

### 3.3 标准三栏舞台

大多数模板的图表区域：

```css
.stage {
  display: grid;
  grid-template-columns: 220px 1fr 280px;
  gap: 32px;
  align-items: start;
}
```

| 栏           | 宽度  | 内容                 |
| ------------ | ----- | -------------------- |
| **左** | 220px | 图例、过滤器、控件   |
| **中** | 1fr   | 图表本体             |
| **右** | 280px | 详情面板（联动视图） |

对于不需要左栏的图（旭日、矩形树图），用两栏 `1fr 280px`。

### 3.4 构图原则（含格式塔依据）

1. **非对称头部**（`grid: 1fr auto`）创造视觉张力；对称布局显得模板化。
2. **图表周围充裕的负空间**把图表作为"图形"从"背景"中分离——图地分离原则。
3. **垂直节奏**用一致的 14 / 18 / 24 / 32px 间距单位；邻近性把相关控件成组。
4. **细线边框**（`1px solid var(--line)`）替代重盒子，分组而不喧宾夺主。
5. **黄金分割比例**：图表宽:右栏 ≈ 1080:280 ≈ 3.86:1，接近 φ²，视觉舒适。

---

## 4. 交互模式

所有模板支持以下五种交互原语，按图类型选用：

### 4.1 悬停 → 高亮 + 其他变暗（弹出效应）

```javascript
function mouseEnter(event, d){
  const ancestors = new Set(d.ancestors ? d.ancestors() : [d]);
  marks.classed("dim", n => !ancestors.has(n))
       .filter(n => n === d).classed("glow", true);
  tip.classed("on", true);
  updateDetail(d);
}
```

**感知依据：** 通过明度对比的选择性注意。变暗元素降至 18-22% 透明度——既保留上下文，又低到让被高亮元素在 <250ms（预注意阈值）内"弹出"。低于 18% 上下文丢失，高于 25% 弹出不够强。

### 4.2 Tooltip（按需细节）

固定定位，跟随光标，偏移 `+18px, +18px`。内容包含：

- 元素名（Fraunces，较大）
- 路径/面包屑（mono，dim）
- 1-3 项数值（mono，tabular）
- 可选色带（呼应该元素的颜色编码）

样式：背景半透明 + 细线边框 + `backdrop-filter: blur(8px)`。淡入 150ms。

```css
.tip {
  position: fixed;
  pointer-events: none;
  z-index: 99;
  background: rgba(17,20,27,.96);
  border: 1px solid var(--line-2);
  padding: 10px 14px;
  min-width: 170px;
  font-size: 12px;
  box-shadow: 0 20px 40px rgba(0,0,0,.4);
  backdrop-filter: blur(8px);
  opacity: 0;
  transform: translateY(4px);
  transition: opacity .15s, transform .15s;
}
.tip.on { opacity: 1; transform: translateY(0); }
```

### 4.3 点击 → 缩放 / 下钻

适用于层级或可缩放习语（Sunburst、Treemap、Force、Choropleth）。

```javascript
const T_ZOOM = 820;   // 慢到能保持对象恒常性，快到不令人不耐烦
selection.transition()
  .duration(T_ZOOM)
  .ease(d3.easeCubicInOut);
```

### 4.4 联动详情面板

右栏在悬停或点击时更新，展示：

- 选区名称 + 面包屑
- 3 行统计：值 / 占父级比 / 占总体比
- 进度条
- 子组件列表（按值排序）

**为什么需要面板：** 人眼对**角度**与**面积**的精确读数能力弱（误差可达 30%）。详情面板用文字数字补偿这个生理局限。

### 4.5 图例过滤（联动高亮）

左栏图例条响应悬停（高亮该类别的图表元素）和点击（下钻或过滤）。颜色与图表保持一致。

```javascript
legend.selectAll(".legend-row")
  .on("mouseenter", (e, d) => {
    marks.classed("dim", m => m.category !== d.name);
  })
  .on("mouseleave", () => {
    marks.classed("dim", false);
  });
```

---

## 5. 动效

### 5.1 时间常量

```javascript
const T = {
  enter:    700,   // 初始挂载错峰
  hover:    250,   // dim/glow 过渡
  zoom:     820,   // 主要视图变化
  tooltip:  150,   // 淡入
  number:   820,   // 数值插值
  brush:    180,   // 刷选反馈
};
```

### 5.2 标准缓动

| 场景            | 缓动                              | 为什么                     |
| --------------- | --------------------------------- | -------------------------- |
| 入场            | `d3.easeCubicOut`               | 快起缓收，吸引眼球         |
| 缩放过渡        | `d3.easeCubicInOut`             | 对称，显得审慎             |
| 微交互（hover） | `d3.easeQuadOut`                | 低调不抢戏                 |
| 弹性反馈        | `d3.easeBackOut.overshoot(1.1)` | 用于成功态确认（克制使用） |

### 5.3 入场错峰

总是让标记以逐元素延迟方式入场：

```javascript
marks.attr("opacity", 0)
  .transition()
  .delay((d, i) => 30 + i * 12)
  .duration(700)
  .ease(d3.easeCubicOut)
  .attr("opacity", 1);
```

**感知依据：** 共同命运（格式塔）——同步扫入告诉观察者"这些元素属于一组"。同时引导注意力沿数据方向（径向外、左到右、按数值排序等）。

### 5.4 数值插值

交互引起数值变化时，**永远插值，绝不跳变**：

```javascript
selection.transition().duration(820)
  .tween("text", function(){
    const i = d3.interpolateNumber(oldVal, newVal);
    return function(t){
      this.textContent = i(t).toFixed(2);
    };
  });
```

数字跳变破坏对象恒常性，让人怀疑自己看错了；插值过渡保留"这是同一个数据点在变化"的感知。

### 5.5 动效禁忌

- ❌ 无限循环动画（旋转、闪烁）——除非是加载态
- ❌ 自动播放视频/动画——用户没请求
- ❌ 单次动画 > 900ms——超过认知耐受
- ❌ 阻塞交互的动画——动画期间用户应该仍可点击
- ❌ `transform: scale()` 大于 1.05——容易引起晕动

---

## 6. 焦点态与键盘导航

### 6.1 焦点环

所有可交互元素需要清晰的焦点态：

```css
.cell:focus-visible,
.legend-row:focus-visible,
.crumb:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

/* 给 SVG 元素加焦点环（outline 在 SVG 内表现不一致） */
.cell rect:focus-visible {
  stroke: var(--accent);
  stroke-width: 2px;
}
```

### 6.2 键盘导航

主要交互元素加 `tabindex="0"`：

```javascript
cells.attr("tabindex", 0)
  .attr("role", "button")
  .attr("aria-label", d => `${d.data.name}, value ${d.value}`);

cells.on("keydown", (event, d) => {
  if (event.key === "Enter" || event.key === " "){
    event.preventDefault();
    if (d.children) render(d);
  }
});
```

### 6.3 ARIA 标注

- 图表容器：`role="img"` + `aria-label="树状图，2023 年全球碳排放按区域和行业拆分"`
- 可点击单元：`role="button"` + `aria-label` 带名称和数值
- 详情面板：`aria-live="polite"`——内容更新时屏幕阅读器会朗读

---

## 7. 空状态与错误

### 7.1 空数据

当 `data` 为空或所有 `value` 都是 0：

```html
<div class="empty">
  <div class="empty-icon">∅</div>
  <div class="empty-title">没有数据可显示</div>
  <div class="empty-hint">检查数据源，或调整过滤条件</div>
</div>
```

```css
.empty {
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  padding: 80px 24px;
  color: var(--ink-mute);
  text-align: center;
}
.empty-icon { font-size: 48px; opacity: .3; margin-bottom: 16px; font-family: 'Fraunces', serif; }
.empty-title { font-family: 'Fraunces', serif; font-size: 18px; color: var(--ink-dim); margin-bottom: 6px; }
.empty-hint { font-family: 'JetBrains Mono', monospace; font-size: 11px; letter-spacing: .12em; text-transform: uppercase; }
```

### 7.2 数据加载失败

```javascript
try {
  // ... 图表渲染
} catch (err) {
  d3.select("#chart").html(`
    <foreignObject width="100%" height="100%">
      <div class="empty">
        <div class="empty-icon">!</div>
        <div class="empty-title">数据解析失败</div>
        <div class="empty-hint">${err.message}</div>
      </div>
    </foreignObject>
  `);
}
```

---

## 8. 背景氛围

纯色背景显得平板。每个模板叠加两层微妙氛围：

```css
/* 第 1 层：径向光斑，加温度、打破平整 */
body::before {
  content: "";
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  background:
    radial-gradient(800px 600px at 18% 12%, rgba(240,198,116,.06), transparent 60%),
    radial-gradient(900px 700px at 85% 88%, rgba(227,106,74,.05), transparent 60%);
}

/* 第 2 层：分形噪点，加质感、防色带 */
body::after {
  content: "";
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  opacity: .35;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='160' height='160'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 1  0 0 0 0 1  0 0 0 0 1  0 0 0 .04 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  mix-blend-mode: overlay;
}
```

**感知依据：** 图地分离。轻微纹理的低对比度背景比平面背景更可靠地"后退"——图表元素作为"图形"获得突出地位。这是电影摄影常用的"环境光"技巧。

---

## 9. 响应式

`< 1100px` 时折叠为单列：

```css
@media (max-width: 1100px) {
  .stage {
    grid-template-columns: 1fr;
  }
  .frame {
    padding: 32px 24px;
  }
  header.title {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  .meta {
    text-align: left;
  }
}

@media (max-width: 680px) {
  h1 {
    font-size: 32px;
  }
  .frame {
    padding: 24px 16px;
  }
}
```

注意：复杂图（如桑基、平行坐标）在窄屏上几乎无法使用，可以选择隐藏图表 + 显示提示"请在更大屏幕上查看"。

---

## 10. 无障碍核查清单

宣告模板完成前请走一遍。

### 色觉障碍

- [ ] 所有分类色在灰度下可区分（`filter: grayscale(100%)` 测试）
- [ ] 没有信息**仅**靠红绿表达
- [ ] 颜色始终搭配至少一个其他通道（标签、位置、形状）

### 对比度（WCAG AA）

- [ ] 正文 ≥ 4.5:1
- [ ] 大标题 ≥ 3:1
- [ ] 图表标记对背景 ≥ 3:1
- [ ] tooltip 文本 ≥ 4.5:1

### 预注意特征

- [ ] 悬停状态变化在 <250ms 内可察觉（透明度或明度，不是细微色相变化）
- [ ] 选中/聚焦元素毫无疑问是屏幕最亮/最饱和的
- [ ] 强调色用量稀疏，使其使用时能真正弹出

### 动效与运动安全

- [ ] 所有过渡 ≤ 900ms
- [ ] 入场动画不阻塞首次交互（动画期间仍可点）
- [ ] 没有无限循环动画引起晕动症
- [ ] 尊重 `prefers-reduced-motion` 媒体查询（可选但推荐）

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 格式塔原则

- [ ] **邻近性：** 相关控件视觉成组
- [ ] **相似性：** 同类元素至少共享一个视觉属性（颜色或形状）
- [ ] **共同区域：** 面板有清晰边界
- [ ] **连续性：** 阅读顺序由布局方向强化（上→下，左→右，或径向外扩）
- [ ] **图地：** 图表与背景明显分离

### 认知负荷

- [ ] 分类色 ≤ 7-8
- [ ] 字体家族 ≤ 3
- [ ] tooltip 数值 ≤ 3
- [ ] 详情面板 ≤ 1 屏（主要信息无需滚动）

### 键盘与屏幕阅读器

- [ ] 主要交互元素有 `tabindex="0"`
- [ ] 焦点环清晰可见
- [ ] 图表容器有 `role="img"` 和 `aria-label`
- [ ] 详情面板有 `aria-live="polite"`
