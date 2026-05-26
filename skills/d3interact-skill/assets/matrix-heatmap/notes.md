# Matrix Heatmap 矩阵热图 — 使用说明

## 何时使用
数据是 R 行 × C 列的二维表，每个 cell 一个值。行列各代表一个分类维度。最适合看交叉模式、聚类、对角线主导。

典型场景：相关性矩阵、混淆矩阵、共现矩阵、用户-商品评分、基因表达、模型-指标对比表。

## 必填占位符

通用部分：`TITLE / EYEBROW / H1_LEFT / H1_EM / H1_RIGHT / LEDE / META_TITLE / META_SUB / META_DATA / UNIT / DATA`

本图特有：

| 占位符 | 填什么 | 示例 |
|--------|--------|------|
| `{{ROOT_NAME}}` | 详情面板默认状态的名字 | `Correlation matrix` |
| `{{COLOR_SCHEME}}` | `"sequential"` 或 `"diverging"` | sequential：单色梯度（值都同号）。diverging：发散色（值有正有负，围绕 0） |

## 数据结构

```javascript
{
  "rows": ["R1", "R2", "R3"],        // 行名（按显示顺序）
  "cols": ["C1", "C2", "C3", "C4"],   // 列名
  "values": [                          // 二维数组：values[行索引][列索引]
    [0.82, 0.21, 0.55, 0.33],
    [0.14, 0.91, 0.30, 0.66],
    [0.62, 0.40, 0.71, 0.18]
  ]
}
```

允许 `null` 表示缺失值（会渲染为深灰底色）。

## 交互

| 操作 | 结果 |
|------|------|
| 悬停单元格 | tooltip + 该行该列以外的格变暗 + 行列标签高亮 + 行列边缘条变色 |
| 悬停行标签 | 整行高亮（其他变暗） + 详情面板显示该行 top 5 |
| 悬停列标签 | 整列高亮 + 详情面板显示该列 top 5 |
| 点击 "Sort rows / cols" 按钮 | 重排矩阵（Original / By sum / By cluster） |

**三种排序模式：**

- **Original**：保留用户给的原始顺序
- **By sum**：按行/列总和降序——大值排在前面，一眼看主导项
- **By cluster**：按"重心索引"排序（值加权平均索引）——结构相似的行/列会被聚到一起，矩阵会显示出**块对角**或**带状**模式（如果数据有聚类结构）

## 颜色刻度

- **sequential**：深背景 → 琥珀，编码"大小"。适合非负数据（评分、计数、概率）。
- **diverging**：蓝 → 中性 → 红，编码"方向+大小"。适合有正负的数据（相关系数、差值、Z-score）。**绝对不要**用红 → 绿（色盲不安全）。

刻度范围自动从数据极值推算；diverging 会对称围绕 0。

## 边缘条带

矩阵右侧和下方各有一条小条带，显示每行/每列的总和。当悬停某行/某列时对应条带会高亮——这是给"颜色辨别"加一个**位置编码**冗余，弥补颜色梯度精度不足。

## 定制需求

| 用户需求 | 怎么做 |
|---------|--------|
| 改主色（如蓝色梯度） | 改 `colorScale` 的 `interpolateRgb("#1c1e26", "#f0c674")` 终点色为 `#6a9bd1` 等 |
| 用其他 D3 色板 | 替换 `colorScale.interpolator(d3.interpolateViridis)`（顺序）或 `d3.interpolatePuOr`（发散） |
| 在格子里显示数值 | render() 末尾给 `cellSel` 加 `<text>` 节点，文字色取 `c.value` 亮暗自动反色 |
| 隐藏边缘条带 | 删除 `row-summaries` 和 `col-summaries` 两个 `<g>` 块 |
| 禁用排序 | 删除 `.toolbar` 里的 `.sort-group`，移除 `d3.selectAll(".sort-btn").on("click", ...)` |
| 行列名太长重叠 | 把 `font-size` 调小到 `9px`；或在 CSS 里加 `text-overflow: ellipsis`（需配合 `foreignObject`） |
| 单元格间距更大 | 调 `xScale/yScale` 的 `.padding(0.04)` 到 `0.1` |

## 限制

- 同屏建议 ≤ 60 × 60 ≈ 3600 个单元格；再大单元格小于 8px，悬停命中精度下降
- 颜色梯度无法精确读数 → 必须保留 tooltip
- 行列名过长会和相邻标签重叠；考虑缩写或旋转角度调整（当前 -45°）
- `cluster` 排序是简易"重心法"，不是真正的层次聚类。如果用户要严格的 hierarchical clustering，需要在数据预处理阶段先跑一遍（如 scipy.cluster），然后把行列已排序的数据传进来，用 `original` 模式显示
- 对角线主导的数据（如混淆矩阵）配合 `By cluster` 排序最能展示结构

## 数据准备建议

如果你的数据是"长表"（每行一条 `{row, col, value}`），用 D3 转换：

```javascript
const longData = [{row:"A", col:"X", value:5}, ...];
const rows = [...new Set(longData.map(d => d.row))];
const cols = [...new Set(longData.map(d => d.col))];
const matrix = rows.map(r =>
  cols.map(c => {
    const found = longData.find(d => d.row === r && d.col === c);
    return found ? found.value : null;
  })
);
const data = { rows, cols, values: matrix };
```
