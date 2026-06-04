---
name: kb-as-memory
description: 三层记忆系统——FTS5全文索引 → 同义扩展语义检索 → 自动归档。让知识库不再是静态仓库，而是可实时检索、自动沉淀的记忆层。
---

# KB as Memory — 知识库即记忆系统

## 架构总览

```
Layer 1: FTS5 全文索引
  定时自动增量更新 | 覆盖整个知识库
  → 快速精确检索，知道「有没有、在哪里」

Layer 2: 语义检索（同义扩展+加权）
  同义词扩展查询 | TF-IDF 重排序
  → 找得到「意思相同但字不同」的内容

Layer 3: 自动归档（会话结束触发）
  分类 → 提取 → 写入 → 更新索引
  → 新知识自动沉淀，无需手动说「记录一下」
```

## Layer 1：FTS5 全文索引

### 索引构建

```bash
# 建索引脚本示例（Python + FTS5）
python3 kb_index.py          # 全量构建
python3 kb_index.py --incremental  # 增量更新
python3 kb_query.py "关键词" --limit 5  # 查询
python3 kb_query.py --stats  # 索引统计
python3 kb_query.py --browse  # 浏览最新文件
```

### 索引覆盖范围

| 来源 | 内容 |
|:-----|:-----|
| KB 本地文件 | 知识库全部 `.md` 文件 |
| Hermes Skills | 已安装的 SKILL 文件 |
| 总计 | 自动发现，无需手动注册 |

### 索引内容

每个KB文件提取：
- 标题（from frontmatter.title 或文件名）
- 类型（from frontmatter.type）
- 学科（from frontmatter.discipline）
- 场景（from frontmatter.scenario）
- 标签（from frontmatter.tags）
- 正文全文

### 自检与维护

每次会话开始时自动检查：
1. 索引 DB 是否存在 → 不存在则全量构建
2. 检查最新修改时间 → 超过阈值则增量更新
3. 调用 `--stats` 确认索引健康

### SQLite 表结构

```sql
CREATE TABLE kb_index (
  slug TEXT PRIMARY KEY,
  title TEXT,
  type TEXT,
  discipline TEXT,
  scenario TEXT,
  status TEXT,
  tags TEXT,
  summary TEXT,
  file_path TEXT,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

FTS5 虚拟表覆盖：title, type, tags, summary。

## Layer 2：语义检索（同义扩展）

### 能力

- **同义词扩展**：专有名词自动联想（如「芒格」→「逆向」/「心智模型」）
- **多查询变体**：原始查询 + 扩展查询，合并去重
- **场景感知优先**：自动匹配场景 → 加权排序
- **TF-IDF 重排序**：标题/路径/标签加权

### 场景感知检索

```
用户查询
   ↓
① 场景匹配 → 触发关键词匹配（长词加权）
   ↓ 匹配成功          ↓ 无匹配
② 场景加权             ③ 全库回退
   (命中场景+5分)        (传统FTS5)
   ↓ 有结果              ↓
④ 输出场景感知结果      ⑤ 输出全库结果
```

### 核心原则

1. **纯关键词匹配，不猜语义** — 宁可漏掉也不误触发
2. **长词加权** — 短通用词（如「收敛」）不会误配无关场景
3. **无匹配回退** — 场景感知返回0结果时自动降级到全库FTS5

## Layer 3：自动归档

### 触发方式

| 触发条件 | 动作 |
|:---------|:-----|
| 用户说「记录一下/记一下/这个有价值」 | 即时写入 |
| 会话结束（用户说「今天就到这」等） | 扫描本轮内容 → 归档 |
| 定时 cron（可选，每夜/每4小时） | 扫描当日会话 → 归档 |

### 归档分类

| 内容特征 | → KB类型 | 目标目录 |
|:---------|:---------|:---------|
| 新方法论、流程、框架 | method | 200-概念与方法/ |
| 新概念、定义 | concept | 200-概念与方法/ |
| 实战分析、决策结论 | case | 400-案例/ |
| 新工具、脚本、配置 | tool | 300-工具与SOP/ |
| 操作步骤、流程 | sop | 300-工具与SOP/ |
| 事实/数据/参数 | fact | 500-事实/ |

### 写入优先级（P0-P5）

| 优先级 | 内容 | 说明 |
|:-------|:-----|:-----|
| P0 | 用户说「记录一下」 | 无条件写入 |
| P1 | 可复用方法论/决策框架 | |
| P2 | 新概念/定义 | |
| P3 | 完整实战案例（背景+推演+结论） | |
| P4 | 新工具/配置（已验证） | |
| P5 | 被纠正过的重要事实 | |

### 写入前检查（三道门）

1. **质量门** — ≥3句实质性内容，否则不写
2. **去重门** — 检查是否已有同主题内容（有则更新旧条目）
3. **确认门** — 自动判断的归档先问用户确认，不替用户决定

### 不写什么（Skip List）

- 纯闲聊/寒暄
- 一次性操作指令（下次用不上）
- 中途失败的尝试（除非教训本身有价值）
- 未确认的推测
- 已有更完整版本的内容（更新旧版，不建新条目）

## 快速上手

```bash
# 1. 安装本 SKILL 后，构建索引目录
mkdir -p ~/kb/{200-概念与方法,300-工具与SOP,400-案例,500-事实}

# 2. 写建索引脚本 kb_index.py（核心逻辑：遍历目录→提取frontmatter→写入FTS5）
#    参考 Layer 1 中的表结构

# 3. 在 Hermes 中加载
/skill kb-as-memory

# 4. 告诉 Agent：「开始用知识库作为我的记忆层」
```

## 依赖

本 SKILL 依赖 `knowledge-base-architecture` 定义的 frontmatter 协议和目录结构。如果尚未安装，先安装 `knowledge-base-architecture`。
