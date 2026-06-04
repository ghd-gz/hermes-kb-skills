---
name: pptx-to-html-mece
description: 将PowerPoint(.pptx)文件转化为视觉效果一致的独立HTML文件。核心：PPTX是坐标系统，直接读坐标而非靠视觉猜。MECE三层穿透（Master→Layout→Slide）+ 多样本交叉对比。
---

# PPTX→HTML 精确复刻 · MECE三层穿透法

## 核心原则

PPTX→HTML 不是"视觉再现"，而是**坐标映射问题**。每页PPT在OpenXML坐标系中有精确的定位数据，直接读取这些数值映射到HTML，比任何视觉分析都精确。

```
px = EMU / 914400 × 96
标准PPTX 16:9 = 1280 × 720 px
```

## 三步穿透（MECE）

```
Step 1: Master层（幻灯片母版）
  → 全局背景、主题色、跨所有slide的占位符

Step 2: Layout层（版式）
  → 每类页面的固定骨架（标题栏图、占位符位置）
  ⚠️ 最容易漏掉的层！标题栏图在Slide层是看不到的

Step 3: Slide层（幻灯片）
  → 每页实际内容（文字、图片、分组形状）
```

## 多页交叉对比

目录页/内容页等多页重复布局，必须对比**至少2-3个样本**，区分"固定结构"和"可变内容"：

```python
from collections import defaultdict
layouts = defaultdict(list)
for idx, slide in enumerate(prs.slides):
    sig = ','.join(str(sh.shape_type).split('.')[-1] for sh in slide.shapes)
    layouts[sig].append(idx)
# 相同sig=相同布局，同组内比较shape差异
```

## 四类模板速查

| 模板 | 来源 | 固定布局 | 可变内容 |
|------|------|---------|---------|
| 封面 | 标题幻灯片 | 全幅背景+左上logo+标题居中 | 标题文字 |
| 目录 | 2-4页交叉对比 | header栏+左图标+4项 | 每项文字 |
| 内容页 | 多页交叉对比 | header栏+标题+副标题+内容区 | 标题/副标题/正文/图 |
| 尾页 | 空白布局 | 全幅背景+"感谢聆听" | 无 |

## 坐标提取代码

```python
from pptx import Presentation
prs = Presentation('input.pptx')
slide = prs.slides[0]

for shape in slide.shapes:
    # EMU→px (96dpi)
    l_px = shape.left / 914400 * 96
    t_px = shape.top / 914400 * 96
    w_px = shape.width / 914400 * 96
    h_px = shape.height / 914400 * 96
    
    # 文本提取
    if hasattr(shape, 'text_frame'):
        for para in shape.text_frame.paragraphs:
            for run in para.runs:
                sz_pt = run.font.size / 12700 if run.font.size else 'inherit'
                name = run.font.name or 'inherit'
                bold = run.font.bold
                try: color = f"#{run.font.color.rgb}"
                except: color = 'inherit'
    
    # 图片提取（Slide层）
    if shape.shape_type == MSO_SHAPE_TYPE.PICTURE:
        blob = shape.image.blob  # base64嵌入
```

### Layout层图片提取

```python
for sh in slide.slide_layout.shapes:
    if sh.shape_type == MSO_SHAPE_TYPE.PICTURE:
        for blip in sh._element.iter('{http://schemas.openxmlformats.org/drawingml/2006/main}blip'):
            rid = blip.get('{http://schemas.openxmlformats.org/officeDocument/2006/relationships}embed')
            if rid and rid in slide.slide_layout.part.rels:
                tp = slide.slide_layout.part.rels[rid].target_part
                blob = tp.blob  # Layout层的图片
```

## 图片处理

| PPTX原格式 | HTML推荐格式 | 原因 |
|-----------|------------|------|
| JPEG | JPEG q85 | 无透明度 |
| PNG | PNG(有透明度)或JPEG(无) | 减小体积 |
| TIFF | PNG | 浏览器不支持TIFF |

## 10大常见陷阱

| # | 陷阱 | 后果 | 解决 |
|---|------|------|------|
| 1 | 只扫Slide层 | 漏Layout层标题栏图 | 必扫layout.shapes |
| 2 | vision猜坐标 | 位置偏差50%+ | 读PPTX精确EMU值 |
| 3 | 只取单样本 | 误把可变当固定 | 至少2-3个对比 |
| 4 | 只读第一段 | 漏多段落文本 | 遍历全部paragraphs |
| 5 | 忽略GROUP | 漏复杂布局 | 递归group.shapes |
| 6 | PNG不转换 | 文件巨大 | 无透明度→JPEG |
| 7 | 大图原分辨率 | HTML过大 | 降采样到显示尺寸 |
| 8 | 不读封面坐标 | 标题位置偏 | 从PPTX读精确top |
| 9 | TIFF直接引用 | 浏览器不显示 | 转PNG |
| 10 | 主题色scheme | 颜色不对 | 降级为#1a1a1a |
