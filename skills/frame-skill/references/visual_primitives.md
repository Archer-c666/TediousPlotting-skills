# 图形语汇库 Visual Primitives

让架构图脱离"AI 圆角矩形拼图"的关键，就是用对图种的视觉语汇。
本文件给出一组**可直接拷贝、改坐标就能用的 SVG 片段**。
每个片段都配了：**用在什么场景** + **完整 SVG 代码** + **怎么改尺寸**。

---

## 1. 透视特征图（CNN feature map）

**用于**：神经网络架构图里表示卷积层输出的特征图。CNN/UNet/分割/检测论文几乎必用。
**为什么**：CV 论文里几乎不用平面矩形表示 feature map。用带透视的层叠平面，一眼能看出"这是张量，有通道维度"。

```xml
<!-- 透视特征图：在 (x, y) 位置画一个尺寸 w×h 的特征图，通道数 C 由层叠数表示 -->
<g transform="translate(400, 200)">
  <!-- 后两层（深度感） -->
  <rect x="12" y="-12" width="100" height="100" rx="2"
        fill="#7BA7D9" opacity="0.35"/>
  <rect x="6"  y="-6"  width="100" height="100" rx="2"
        fill="#7BA7D9" opacity="0.65"/>
  <!-- 最前面一层（实色） -->
  <rect x="0"  y="0"   width="100" height="100" rx="2"
        fill="#7BA7D9" stroke="#5A89BC" stroke-width="1.2"/>
  <!-- 通道数标注 -->
  <text x="50" y="118" text-anchor="middle" font-size="12" fill="#4B5563">
    64 × 64 × 128
  </text>
</g>
```

**改法**：调 transform 改位置；调 width/height 改尺寸；偏移量 6/12 px 不要变（变了就不像透视了）。通道数越多可以叠 4-5 层（每层多偏移 6px）。

---

## 2. 多层堆叠（Transformer × N、ResNet block × N）

**用于**：Transformer 层堆叠、Residual Block 重复 N 次、任何"这个模块要重复 N 次"的场景。

```xml
<g transform="translate(600, 300)">
  <!-- 后景层 -->
  <rect x="8"  y="8"  width="160" height="80" rx="8"
        fill="#B197D6" opacity="0.4"/>
  <rect x="4"  y="4"  width="160" height="80" rx="8"
        fill="#B197D6" opacity="0.7"/>
  <!-- 主层 -->
  <rect x="0"  y="0"  width="160" height="80" rx="8"
        fill="#B197D6" stroke="#8E72B5" stroke-width="1.2"/>
  <text x="80" y="46" text-anchor="middle" font-size="15"
        font-weight="600" fill="#FFFFFF">Transformer Block</text>
  <!-- × N 标记，画在右上角外侧 -->
  <text x="175" y="20" font-size="14" font-weight="bold" fill="#1F2937">× 12</text>
</g>
```

---

## 3. 回环箭头（迭代/反馈/训练循环）

**用于**：style_05 循环图、控制系统反馈环、训练 loop 里的"梯度回传"。
**为什么**：普通灰色细箭头表达不出"这是关键循环"的语义。回环必须粗、必须用 accent 色。

```xml
<!-- 在 <defs> 里先定义 accent 色的箭头 -->
<marker id="arrow_accent" viewBox="0 0 10 10" refX="9" refY="5"
        markerWidth="8" markerHeight="8" orient="auto-start-reverse">
  <path d="M0,0 L10,5 L0,10 z" fill="#DC2626"/>
</marker>

<!-- 回环本体：从 (700, 400) 弧线绕回 (300, 400)，弧顶在 (500, 200) -->
<path d="M 700 400 Q 500 200 300 400"
      fill="none" stroke="#DC2626" stroke-width="3"
      marker-end="url(#arrow_accent)"/>
<!-- 旁边标签 -->
<text x="500" y="245" text-anchor="middle" font-size="13"
      font-style="italic" fill="#DC2626">iterate (T steps)</text>
```

---

## 4. 数据样例缩略图占位

**用于**：style_02 水平流程、任何 pipeline 图。在两个步骤之间放一个小缩略图代表"这一步的输出长什么样"。
**为什么**：纯箭头连接的 pipeline 图很抽象。加了缩略图就有了"过程感"。

```xml
<!-- 在两个步骤之间画一个 48×48 的占位框 -->
<g transform="translate(300, 250)">
  <rect x="0" y="0" width="48" height="48" rx="4"
        fill="#FFFFFF" stroke="#9CA3AF" stroke-width="1.2"
        stroke-dasharray="3,2"/>
  <text x="24" y="30" text-anchor="middle" font-size="10"
        fill="#9CA3AF" font-style="italic">[sample]</text>
</g>
```

**用户给了真实图就把 rect 换成 `<image href="..." width="48" height="48"/>`**。

---

## 5. 输入图像占位（CV 论文的输入位）

**用于**：图像类论文的"输入"位置。比"一个写着 input 的矩形"高级很多。

```xml
<g transform="translate(80, 200)">
  <!-- 图像框 -->
  <rect x="0" y="0" width="140" height="140" rx="4"
        fill="#F3F4F6" stroke="#6B7280" stroke-width="1.5"/>
  <!-- 模拟图像内容：几个色块 + 一条对角线，暗示"这是一张图" -->
  <rect x="20" y="30" width="40" height="50" fill="#D1D5DB"/>
  <rect x="70" y="50" width="50" height="60" fill="#9CA3AF"/>
  <path d="M 0 140 L 140 0" stroke="#6B7280" stroke-width="0.6" opacity="0.3"/>
  <!-- 尺寸标注 -->
  <text x="70" y="160" text-anchor="middle" font-size="12" fill="#4B5563">
    224 × 224 × 3
  </text>
</g>
```

---

## 6. Q / K / V 三色块拼接（注意力机制）

**用于**：Transformer 注意力模块。把 Q/K/V 画成三个紧邻的小色块，比"一个写着 Attention 的方块"清晰得多。

```xml
<g transform="translate(500, 300)">
  <!-- Q -->
  <rect x="0"  y="0" width="40" height="80" rx="4" fill="#7BA7D9"/>
  <text x="20" y="46" text-anchor="middle" font-size="14"
        font-weight="bold" fill="#FFFFFF">Q</text>
  <!-- K -->
  <rect x="44" y="0" width="40" height="80" rx="4" fill="#88C088"/>
  <text x="64" y="46" text-anchor="middle" font-size="14"
        font-weight="bold" fill="#FFFFFF">K</text>
  <!-- V -->
  <rect x="88" y="0" width="40" height="80" rx="4" fill="#F0A776"/>
  <text x="108" y="46" text-anchor="middle" font-size="14"
        font-weight="bold" fill="#FFFFFF">V</text>
  <!-- 整组标签 -->
  <text x="64" y="100" text-anchor="middle" font-size="12" fill="#4B5563">
    Multi-Head Attention
  </text>
</g>
```

---

## 7. Accent 高亮行（消融表 / 对比矩阵的 "Ours"）

**用于**：style_07 网格矩阵。一定要让"Ours"那一行/列视觉上跳出来，否则就是普通表格。

```xml
<!-- 普通行 -->
<rect x="240" y="200" width="215" height="92" fill="#FFFFFF" stroke="#E5E7EB"/>
<text x="347" y="252" text-anchor="middle" font-size="14" fill="#1F2937">72.4</text>

<!-- "Ours" 高亮行（accent 色） -->
<rect x="240" y="292" width="215" height="92"
      fill="#FEE2E2" stroke="#DC2626" stroke-width="2"/>
<text x="347" y="344" text-anchor="middle" font-size="14"
      font-weight="bold" fill="#DC2626">78.6</text>
<!-- 行标签也加粗 -->
<text x="120" y="344" text-anchor="middle" font-size="14"
      font-weight="bold" fill="#DC2626">Ours</text>
```

---

## 8. 图标库（生物/医学/化学常用，代替纯矩形）

**用于**：Warm Bio 调色板下的图。用真实示意图元代替部分纯矩形，让图脱离 "AI 风"。
全部用 SVG 图元绘制，无任何字体依赖。

### 试管（sample tube）
```xml
<g transform="translate(100, 200)">
  <rect x="0" y="0" width="24" height="60" rx="2"
        fill="#FFFFFF" stroke="#C97B5C" stroke-width="1.5"/>
  <rect x="0" y="35" width="24" height="25"
        fill="#EED2C2"/>
  <ellipse cx="12" cy="60" rx="12" ry="4" fill="#C97B5C"/>
  <rect x="-2" y="-4" width="28" height="6" rx="1" fill="#C97B5C"/>
</g>
```

### 细胞（圆 + 核）
```xml
<g transform="translate(200, 200)">
  <circle cx="0" cy="0" r="28" fill="#D8E2C9" stroke="#8AA86B" stroke-width="1.5"/>
  <circle cx="6" cy="-4" r="10" fill="#8AA86B"/>
</g>
```

### 数据库（圆柱）
```xml
<g transform="translate(300, 200)">
  <ellipse cx="30" cy="0" rx="30" ry="8" fill="#4A6FA5"/>
  <rect x="0" y="0" width="60" height="60" fill="#4A6FA5"/>
  <ellipse cx="30" cy="60" rx="30" ry="8" fill="#4A6FA5"/>
  <ellipse cx="30" cy="0" rx="30" ry="8" fill="none" stroke="#FFFFFF" stroke-width="1"/>
</g>
```

### 云（云服务）
```xml
<g transform="translate(400, 200)">
  <path d="M 20 30 Q 0 30 0 20 Q 0 8 12 8 Q 14 0 28 0 Q 42 0 44 12
           Q 58 12 58 22 Q 58 30 48 30 Z"
        fill="#DCE7F5" stroke="#7BA7D9" stroke-width="1.5"/>
</g>
```

### 大脑（简化）
```xml
<g transform="translate(500, 200)">
  <ellipse cx="30" cy="30" rx="28" ry="24" fill="#FAD6D6" stroke="#E89090" stroke-width="1.5"/>
  <path d="M 30 8 Q 30 50 30 52" stroke="#E89090" stroke-width="1.2" fill="none"/>
  <path d="M 15 20 Q 25 30 15 40" stroke="#E89090" stroke-width="1" fill="none"/>
  <path d="M 45 20 Q 35 30 45 40" stroke="#E89090" stroke-width="1" fill="none"/>
</g>
```

### 用户/角色（用于泳道左侧标签）
```xml
<g transform="translate(80, 200)">
  <circle cx="0" cy="-10" r="8" fill="#7BA7D9"/>
  <path d="M -14 14 Q -14 0 0 0 Q 14 0 14 14 Z" fill="#7BA7D9"/>
</g>
```

---

## 9. 步骤序号徽章（pipeline / 流程图）

**用于**：水平流程、泳道，给每个步骤一个序号。比纯文字"Step 1"显眼。

```xml
<g transform="translate(60, 60)">
  <circle cx="0" cy="0" r="20" fill="#7BA7D9"/>
  <text x="0" y="6" text-anchor="middle" font-size="18"
        font-weight="bold" fill="#FFFFFF">1</text>
</g>
```

---

## 10. 特殊符号的 SVG 替代（避免豆腐框）

cairosvg 渲染时，★ ✓ ✗ ① ↻ 这类符号经常变豆腐框。用下面的 SVG 图元代替：

### 加号 +（残差连接）
```xml
<g transform="translate(800, 300)">
  <circle cx="0" cy="0" r="14" fill="#FAD6D6" stroke="#E89090" stroke-width="2"/>
  <path d="M -7 0 L 7 0 M 0 -7 L 0 7" stroke="#B91C1C" stroke-width="2.5"/>
</g>
```

### 乘号 ×（点乘/拼接）
```xml
<g transform="translate(800, 400)">
  <circle cx="0" cy="0" r="14" fill="#FFE0CC" stroke="#F0A776" stroke-width="2"/>
  <path d="M -6 -6 L 6 6 M -6 6 L 6 -6" stroke="#C2410C" stroke-width="2.5"/>
</g>
```

### 对勾 ✓
```xml
<g transform="translate(800, 500)">
  <circle cx="0" cy="0" r="14" fill="#D6EBD6" stroke="#88C088" stroke-width="2"/>
  <path d="M -7 0 L -2 5 L 7 -5" fill="none" stroke="#15803D" stroke-width="2.5"
        stroke-linecap="round" stroke-linejoin="round"/>
</g>
```

### 叉号 ✗
```xml
<g transform="translate(800, 600)">
  <circle cx="0" cy="0" r="14" fill="#FAD6D6" stroke="#E89090" stroke-width="2"/>
  <path d="M -6 -6 L 6 6 M -6 6 L 6 -6" stroke="#B91C1C" stroke-width="2.5"
        stroke-linecap="round"/>
</g>
```

### 圆圈数字 ① ② ③
直接用 circle + text 数字代替（注意：文字数字 1, 2, 3 是 ASCII，不会变豆腐框，可以放心用）：
```xml
<g transform="translate(800, 700)">
  <circle cx="0" cy="0" r="14" fill="#7BA7D9"/>
  <text x="0" y="5" text-anchor="middle" font-size="16"
        font-weight="bold" fill="#FFFFFF">1</text>
</g>
```

### 循环箭头 ↻
用一个带箭头的圆弧代替：
```xml
<g transform="translate(800, 800)">
  <path d="M 10 -8 A 12 12 0 1 1 -10 -8"
        fill="none" stroke="#4B5563" stroke-width="2"/>
  <path d="M -10 -8 L -14 -4 L -10 -12 Z" fill="#4B5563"/>
</g>
```

---

## 11. 输入/输出张量形状标注

**用于**：神经网络架构图，在层与层之间标注张量形状变化。

```xml
<!-- 在一根连接线上方放形状标注 -->
<text x="500" y="195" text-anchor="middle" font-size="12"
      fill="#4B5563" font-style="italic">[B, 256, 32, 32]</text>
<line x1="450" y1="220" x2="550" y2="220" stroke="#6B7280" stroke-width="2"
      marker-end="url(#arrow)"/>
```

---

## 怎么挑

不是每张图都要用上面所有片段。按图种挑 2-3 个用就够了：

- **CNN/CV 论文** → 必用 1（透视特征图）+ 5（输入图像占位）+ 10 的加号（残差时）
- **Transformer/LLM** → 必用 2（堆叠层 × N）+ 6（QKV 三色块）
- **训练循环 / 控制系统** → 必用 3（粗 accent 回环）
- **数据 pipeline** → 必用 4（缩略图）+ 9（序号徽章）
- **系统架构** → 必用 8 中的数据库/云/用户图标
- **生物/医学** → 必用 8 中的试管/细胞/大脑图标
- **对比矩阵** → 必用 7（accent 高亮行）
- **泳道流程** → 必用 8 中的用户图标 + 9（序号徽章）

**记住：哪怕只引入 1-2 个图种专属片段，图也会立刻"像那个领域的论文图"，而不是"通用 AI 架构图"。**
