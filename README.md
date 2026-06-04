# Hermes KB Skills — 知识库技能包

让 Hermes Agent 拥有结构化的知识管理能力 + 专业PPT制作能力。

## 安装方式

### 方式一：通过 GitHub Tap（推荐）

```bash
hermes skills tap add https://github.com/ghd-gz/hermes-kb-skills
hermes skills install knowledge-base-architecture
hermes skills install guizang-ppt-skill
```

### 方式二：直接 URL 安装

```bash
# 知识库
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/knowledge-base-architecture/SKILL.md
# PPT
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/guizang-ppt-skill/SKILL.md
```

### 方式三：手动部署

将 `skills/` 目录下的 SKILL.md 复制到 `~/.hermes/skills/` 对应目录，重启会话后可用。

## 技能一览（更新）

| 技能 | 分类 | 用途 | 独立可用 |
|:-----|:----|:-----|:--------:|
| **knowledge-base-architecture** | 知识库 | 人与AI共享知识库的完整架构 | ✅ |
| **kb-as-memory** | 知识库 | 三层记忆系统——FTS5→语义检索→自动归档 | ❌ 依赖KB架构 |
| **scene-behavior-core** | 元模式 | 决策树路由 + 六条核心行为铁律 | ✅ |
| **guizang-ppt-skill** | PPT制作 | 网页级横向翻页HTML演示文稿，替代Quarto/Pandoc | ✅ |
| **ppt-making-workflow** | PPT制作 | 8步完整管道——需求对齐→内容组织→交付归档 | ❌ 依赖guizang-ppt-skill |
| **pptx-to-html-mece** | PPT制作 | PPTX文件精确复刻为HTML，三层穿透读坐标 | ✅ |

## PPT制作技能

### guizang-ppt-skill

基于归藏的网页PPT模板，生成横向翻页HTML演示文稿。双风格（电子杂志风/瑞士国际主义风），8种表达模型，ECharts+Mermaid图表集成，单文件自包含交付。

核心特点：
- **两种风格**：叙事型（杂志风）+ 数据型（瑞士风）
- **8种表达模型**：PREP/RIDE/SCQA/STAR/空雨伞/FIRE/SCRTV/5W2H，每页可不同
- **图表双引擎**：ECharts（数据图表）+ Mermaid.js（流程图/架构图）
- **内容完整性铁律**：所有数字/名称/百分比全部保留
- **单文件交付**：图片base64内嵌，双击即看，零依赖

### ppt-making-workflow

PPT制作的完整工作流管道，从需求对齐到交付归档。8步流程 + 硬性Checkpoint节点（Plan/Content/Build/Quality）。

### pptx-to-html-mece

将现有PPTX文件精确复刻为HTML。核心方法：
- **MECE三层穿透**：Master(母版)→Layout(版式)→Slide(页面)
- **坐标映射**：读PPTX EMU坐标直接转CSS px，不用vision猜
- **Layout层图片**：从版式层提取标题栏图等容易被漏掉的元素
- **10大陷阱清单**：已踩过的坑全记录

## 安装方式

```bash
# GitHub Tap（推荐）
hermes skills tap add https://github.com/ghd-gz/hermes-kb-skills

# 安装知识库技能
hermes skills install knowledge-base-architecture
hermes skills install kb-as-memory
hermes skills install scene-behavior-core

# 安装PPT技能
hermes skills install guizang-ppt-skill
hermes skills install ppt-making-workflow
hermes skills install pptx-to-html-mece

# 或直接URL安装
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/guizang-ppt-skill/SKILL.md
```

## 快速开始（PPT）

1. 安装 `guizang-ppt-skill`
2. 在Hermes中加载：`/skill guizang-ppt-skill`
3. 告诉Agent：「帮我做一个PPT，给XX看的，讲XX内容」
4. Agent会按9步工作流产出：需求对齐→内容组织→表达匹配→模板选择→生成→交付

## License

MIT

