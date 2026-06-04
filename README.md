# Hermes KB Skills — 知识库技能包

让 Hermes Agent 拥有结构化的知识管理能力——场景驱动的读写、全文检索记忆、行为模式约束。

## 安装方式

### 方式一：通过 GitHub Tap（推荐）

```bash
hermes skills tap add https://github.com/ghd-gz/hermes-kb-skills
hermes skills install knowledge-base-architecture
hermes skills install kb-as-memory
hermes skills install scene-behavior-core
```

### 方式二：直接 URL 安装

```bash
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/knowledge-base-architecture/SKILL.md
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/kb-as-memory/SKILL.md
hermes skills install https://raw.githubusercontent.com/ghd-gz/hermes-kb-skills/main/skills/scene-behavior-core/SKILL.md
```

### 方式三：手动部署

将 `skills/` 目录下的 SKILL.md 复制到 `~/.hermes/skills/` 对应目录，重启会话后可用。

## 技能一览

| 技能 | 用途 | 独立可用 |
|:-----|:-----|:--------:|
| **knowledge-base-architecture** | 人与AI共享知识库的完整架构——目录结构、frontmatter 元数据协议、场景驱动读写、6类知识分类体系 | ✅ |
| **kb-as-memory** | 三层记忆系统——FTS5全文索引 → 同义扩展检索 → 自动归档，让知识库变成可实时检索的记忆层 | ❌ 依赖 KB 架构 |
| **scene-behavior-core** | 场景体系的元模式——决策树路由 + 六条核心行为铁律，规范 Agent 在任何场景下的行为范式 | ✅ |

## 快速开始

1. 安装 `knowledge-base-architecture` 后，在 Hermes 会话中加载：

```
/skill knowledge-base-architecture
```

2. 告诉 Agent：「我们来搭建知识库」，它会引导你完成初始化。

3. 根据自己的需求，可以：
   - 从 `knowledge-base-architecture` 的 frontmatter 协议开始，给已有文档加元数据
   - 加载 `scene-behavior-core` 让 Agent 遵守核心行为铁律
   - 继续安装 `kb-as-memory` 获得全文检索能力

## 核心设计理念

### 一份数据，两种观察

知识库是一组 `.md` 文件。人可以按目录浏览，Agent 通过 frontmatter 元数据检索。同一份文件，两种使用方式。

### 场景驱动

知识不是平铺的，是按「场景」组织的。写时按「这是概念/方法/案例/工具/事实/SOP」分类存入对应目录，读时按「当前在做什么场景」触发检索。

### 行为元模式

`scene-behavior-core` 定义了所有场景通用的六条行为铁律——显化假设、管理歧义、敢于反对、执行简单性、维持范围纪律、验证不假设。这是从 addyosmani/agent-skills 借鉴并改造的元模式。

## 自定义

所有技能都是开源的。你可以：
- Fork 这个仓库定制自己的 SKILL 包
- 修改 frontmatter 协议中的 discipline 列表（当前8个元学科）
- 扩展场景路由表匹配你自己的业务场景

## License

MIT
